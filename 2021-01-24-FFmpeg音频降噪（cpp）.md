---
layout: post
title: FFmpeg音频降噪（cpp）
subtitle: FFmpeg
author: Kingtous
date: 2021-01-24 18:40:58
header-img: img/about-bg.jpg
catalog: True
tags:
- 音频处理
---

## FFmpeg音频降噪功能的使用

> 网上关于FFmpeg的使用实在太少，这里自己摸索了一下

此处对pcm_s16le，单声道，256kbit/s的音频进行高低通降噪
> 除下高低通，还有lv2，afftdn等，可以参考官网

为了通用性，此处直接读取相关数据到unsigned char*数组中，并直接构造AVCodecContext。

#### 创建音频滤镜图

在FFmpeg里，降噪也是一种滤镜，要想使用该滤镜，需要构造一个滤镜图。

图的起点和终点分别为abuffer和abuffersink。在构造后使用avfilter_graph_parse_ptr传入filter_desr完成图的构造。filter_desr为滤镜效果字符串。

此处使用`"highpass=f=200,lowpass=f=3000"`作为filter_desr，为高低通滤波器，200-3000hz为人声区域

- `:`为同级分隔符，可以对配置传入多个参数，`,`可以另起滤镜配置

```c++
codec = avcodec_find_decoder(AV_CODEC_ID_PCM_S16LE);
    dec_ctx = avcodec_alloc_context3(codec);
    dec_ctx->sample_rate = 16000;
    dec_ctx->sample_fmt = AV_SAMPLE_FMT_S16;
    dec_ctx->channel_layout = AV_CH_LAYOUT_MONO;
    dec_ctx->channels = 1;
    dec_ctx->bit_rate = 256000;
    int ret = avcodec_open2(dec_ctx, codec, NULL);
    if (ret < 0) {
        LOGE("no decoder!!!");
        return;
    }

    char args[512];
    snprintf(args, sizeof(args),
             "sample_rate=%d:sample_fmt=%s:channel_layout=0x%" PRIx64,
             dec_ctx->sample_rate,
             av_get_sample_fmt_name(dec_ctx->sample_fmt), dec_ctx->channel_layout);

    filter_graph = avfilter_graph_alloc();
    if (!filter_graph) {
        LOGE("alloc failed");
        return;
    }
    // 获取对应的filter
    src = avfilter_get_by_name("abuffer");
    sink = avfilter_get_by_name("abuffersink");
    if (!src || !sink) {
        LOGE("no highpass_filter and lowpass_filter!!!");
    }
    ret = avfilter_graph_create_filter(&src_context, src, "in", args, NULL,
                                       filter_graph);
    if (ret < 0) {
        LOGE("fail to create highpass filter.");
    }
    static const enum AVSampleFormat out_sample_fmts[] = {AV_SAMPLE_FMT_S16};
    static const int64_t out_channel_layouts[] = {AV_CH_LAYOUT_MONO, -1};
    static const int out_sample_rates[] = {16000, -1};
    ret = avfilter_graph_create_filter(&sink_context, sink, "out", NULL,
                                       NULL, filter_graph);
    if (ret < 0) {
        LOGE("fail to create sink");
    }

    ret = av_opt_set_int_list(sink_context, "sample_fmts", out_sample_fmts, -1,
                              AV_OPT_SEARCH_CHILDREN);
    ret = av_opt_set_int_list(sink_context, "channel_layouts", out_channel_layouts, -1,
                              AV_OPT_SEARCH_CHILDREN);
    ret = av_opt_set_int_list(sink_context, "sample_rates", out_sample_rates, -1,
                              AV_OPT_SEARCH_CHILDREN);

    inputs->name = av_strdup("in");
    inputs->filter_ctx = src_context;
    inputs->pad_idx = 0;
    inputs->next = NULL;

    outputs->name = av_strdup("out");
    outputs->filter_ctx = sink_context;
    outputs->pad_idx = 0;
    outputs->next = NULL;

    ret = avfilter_graph_parse_ptr(filter_graph, filter_desr, &outputs, &inputs, NULL);
    if (ret < 0) {
        LOGE("fail to parse graph.");
    }
    ret = avfilter_graph_config(filter_graph, NULL);
    if (ret < 0) {
        LOGE("fail to config graph.");
    }
```


#### 逻辑代码
```c++
unsigned char *DenoiseManager::denoise_audio(unsigned char *data, int length) {
    AVPacket* packet = av_packet_alloc();
    AVFrame *frame = av_frame_alloc();
    AVFrame *filt_frame = av_frame_alloc();
    // 设置frame
    frame->channel_layout = AV_CH_LAYOUT_MONO;
    frame->sample_rate = 16000;
    frame->format = AV_SAMPLE_FMT_S16;

    auto * denoise_buf = (unsigned char*) malloc(sizeof(char)*length);
    int denoise_index = 0;
    packet->data = data;
    packet->size = length;
    packet->flags = 0;
    packet->duration = 0;
    packet->pos = 0;
    avcodec_flush_buffers(dec_ctx);
    int ret = avcodec_send_packet(dec_ctx,packet);
    auto data_size = av_get_bytes_per_sample(AV_SAMPLE_FMT_S16);
    if (ret < 0){
        return data;
    }
    while (ret >= 0){
        ret = avcodec_receive_frame(dec_ctx,frame);
        if (ret == AVERROR(EAGAIN)){
            continue;
        }
        if (ret == AVERROR_EOF)
            return data;
        ret = av_buffersrc_add_frame_flags(src_context,frame,AV_BUFFERSRC_FLAG_NO_CHECK_FORMAT); // 不检查格式，加快速度
        if (ret < 0){
            return data;
        }
        while(true){
            ret = av_buffersink_get_frame(sink_context,filt_frame);
            if (ret == AVERROR(EAGAIN)){
                if(denoise_index == length){
                    break;
                }
                continue;
            }
            if (ret == AVERROR_EOF){
                break;
            }
            if (ret < 0){
                return data;
            }
            for (int i = 0; i < filt_frame->nb_samples; i++)
                for (int ch = 0; ch < filt_frame->channels; ch++){
                    memcpy(denoise_buf + denoise_index,filt_frame->data[ch] + data_size * i,data_size);
                    denoise_index += data_size;
                }
            av_frame_unref(filt_frame);
        }
        av_frame_free(&frame);
        av_frame_free(&filt_frame);
        // 重新alloc avframe帧
        filt_frame = av_frame_alloc();
        frame = av_frame_alloc();
    }
    if (frame){
        av_frame_free(&frame);
    }
    if (filt_frame){
        av_frame_free(&filt_frame);
    }
    av_packet_unref(packet);
    av_packet_free(&packet);
    return denoise_buf;
}
```