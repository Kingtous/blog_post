---
layout: post
title: 从给Android APP集成FFMPEG谈JNI的使用
subtitle: Android JNI Usage
author: Kingtous
date: 2020-12-20 13:10:57
header-img: img/about-bg-walle.jpg
catalog: True
tags:
- Android
- JNI
---

####  编译FFMPEG为.so动态链接库

[见上期](https://kingtous.cn/2020/12/07/%E5%AE%89%E5%8D%93%E4%B8%8B%E7%BC%96%E8%AF%91%E5%8F%8A%E4%BF%AE%E5%89%AAFFMPEG%E6%8F%92%E4%BB%B6/)

#### 给Android项目添加CMake支持

- 在`build.gradle`的 `defaultconfig`下添加

  - ```
    externalNativeBuild {
    			abiFilters "arm64-v8a"
    		}
    }
    ```

    - 其中可以使用abiFilters进行cpu架构的筛选

- 在`build.gradle`中的`android`下添加externalNativeBuild，指定cmake配置文件路径

  - ```
    externalNativeBuild {
            cmake {
                path "CMakeLists.txt"
                version "3.10.2"
            }
    }
    ```

  - 建议如上，配置一个与build.grade同级目录下的CMakeLists.txt，方便后期进行总集成，该文件使用`add_subdirectory`进行模块化引入

    - ```cmake
      cmake_minimum_required(VERSION 3.4.1)
      
      ADD_SUBDIRECTORY(src/main/cpp/decoder)
      ```

#### 使用CMake集成.so文件

- 在上述subdirectory下引入.so文件、include文件夹，建立子CMakeLists.txt

  - 建立多的使用宏，不建议直接写绝对路径

  - 通过`include_directories`、`add_library`、`set_target_properties`、`link_target_libraries`添加FFMPEG依赖。通过`aux_source_directory`添加目标目录下的所有source c/cpp文件。下面是一个例子

    ```cmake
    cmake_minimum_required(VERSION 3.4.1)
    
    #解码依赖的必要静态库及头文件目录，头文件路径也可以方便IDE代码提示
    include_directories(        "${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/include/${ANDROID_ABI}"
    )
    
    add_library( avutil
           SHARED
           IMPORTED )
    set_target_properties( avutil
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libavutil.so )
    
    add_library( swresample
           SHARED
           IMPORTED )
    set_target_properties( swresample
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libswresample.so )
    add_library( avcodec
           SHARED
           IMPORTED )
    set_target_properties( avcodec
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libavcodec.so )
    add_library( avfilter
           SHARED
           IMPORTED)
    set_target_properties( avfilter
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libavfilter.so )
    add_library( swscale
           SHARED
           IMPORTED)
    set_target_properties( swscale
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libswscale.so )
    add_library( avdevice
           SHARED
           IMPORTED)
    set_target_properties( avdevice
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libavdevice.so )
    add_library( avformat
           SHARED
           IMPORTED)
    set_target_properties( avformat
           PROPERTIES IMPORTED_LOCATION
           ${PROJECT_SOURCE_DIR}/src/main/cpp/decoder/thirdparty/ffmpeg/libs/${ANDROID_ABI}/libavformat.so )
    
    
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11 -fexceptions")
    aux_source_directory(src SRCS)
    
    add_library(myffmpeg SHARED ${SRCS})
    
    #log库被依赖，这里要加上；
    #注意：静态库依赖对顺序有要求，被依赖的放到最后，比如，log是最后被依赖的，放到最后
    target_link_libraries(myffmpeg avcodec avdevice avfilter avformat avutil swresample log)
    ```

#### 编写JNI文件

- Native Java类
  - 文件路径：`cn.kingtous.myffmpegtest.codec.FFMpegCodecManager`
  - 定义方法：`decodeAudio`

```
public native long decodeAudio(String filePath, String dstPath, OnCodecListener listener);
```

- Native C++类
  - 类名一定要和函数名规则匹配
  - 与上期内容相对应的函数内容

```c++
extern "C" JNIEXPORT jlong JNICALL
Java_cn_kingtous_myffmpegtest_codec_FFMpegCodecManager_decodeAudio(JNIEnv *env,
                                                                             jobject,
                                                                             jstring filePath,
                                                                             jstring dstPath,
                                                                             jobject listener) {
    jclass callback = env->GetObjectClass(listener);
    const char *outfilename, *filename;
    const AVCodec *codec;
    AVCodecContext *context = nullptr;
    int ret;
    FILE *f = nullptr, *outfile = nullptr;
    AVPacket *pkt = nullptr;
    AVFrame *decoded_frame = nullptr;
    // 初始化重采样
    struct SwrContext *swr_ctx = nullptr;
    AVFormatContext *formatContext;
    jmethodID onCodeStart = env->GetMethodID(callback, "onCodecStart","()V");
    // 开始筛选出音频index
    int audioIndex = -1;
    try {
        // 开始解码
        env->CallVoidMethod(listener, onCodeStart);
        // 声明次数
        long times = 0;
        long total_size=0;
        long decoded_size= 0;
        int read = 0;
        filename = env->GetStringUTFChars(filePath, JNI_FALSE);
        outfilename = env->GetStringUTFChars(dstPath, JNI_FALSE);
        // 获取音频格式
        formatContext = avformat_alloc_context();
        if (!formatContext) {
            goto end;
        }
        ret = avformat_open_input(&formatContext, filename, NULL, NULL);
        if (ret != 0){
            goto end;
        }
        avformat_find_stream_info(formatContext, nullptr);

        if (formatContext->nb_streams <= 0) {
            goto end;
        }
        for (int i = 0; i < formatContext->nb_streams; ++i) {
            if (formatContext->streams[i]->codecpar->codec_type == AVMediaType::AVMEDIA_TYPE_AUDIO) {
                audioIndex = i;
                break;
            }
        }
        if (audioIndex == -1) {
            goto end;
        }
        codec = avcodec_find_decoder(formatContext->streams[audioIndex]->codecpar->codec_id);

        pkt = av_packet_alloc();

        if (!codec) {
            goto end;
        }

        context = avcodec_alloc_context3(codec);
        if (!context) {
            goto end;
        }
        avcodec_parameters_to_context(context, formatContext->streams[0]->codecpar);

        /* open it */
        if (avcodec_open2(context, codec, NULL) < 0) {
            goto end;
        }

        f = fopen(filename, "rb");
        if (!f) {
            fprintf(stderr, "Could not open %s\n", filename);
            goto end;
        }
        outfile = fopen(outfilename, "wb");
        if (!outfile) {
            av_free(context);
            goto end;
        }
        /* set options */
        context->channel_layout =
                context->channel_layout == 0 ? av_get_default_channel_layout(context->channels)
                                             : context->channel_layout;
        swr_ctx = swr_alloc_set_opts(NULL,
                                     AV_CH_LAYOUT_MONO,
                                     AV_SAMPLE_FMT_S16, 16000,
                                     context->channel_layout,
                                     context->sample_fmt, context->sample_rate, 0, NULL
        );
        if (!swr_ctx) {
            goto end;
        }

        /* initialize the resampling context */
        if ((ret = swr_init(swr_ctx)) < 0) {
            const char *err_message = av_err2str(ret);
            fprintf(stderr, "%s", err_message);
            on_error(env,callback,listener,CODEC_ERROR,err_message);
            goto end;
        }
        // 计算文件大小
        fseek(f,0L,SEEK_END);
        total_size = ftell(f);
        // 回到起点
        fseek(f,0L,SEEK_SET);
        while ((read = av_read_frame(formatContext, pkt)) == 0) {
            if (pkt->stream_index == audioIndex) {
                if (!decoded_frame) {
                    if (!(decoded_frame = av_frame_alloc())) {
                        on_error(env,callback,listener,CODEC_ERROR,"cannot allocate audio frame");
                        goto end;
                    }
                }
                decoded_size += pkt->size;
                times ++;
                if (pkt->size)
                    decode(context, pkt, swr_ctx, decoded_frame, outfile);
                if (times % 30 == 0){
                    on_progress(env,callback,listener,int(decoded_size*100.0/total_size));
                }
            }
        }

        /* flush the decoder */
        pkt->data = NULL;
        pkt->size = 0;
        decode(context, pkt, swr_ctx, decoded_frame, outfile);
        on_success(env,callback,listener,outfilename);
        end:
        if (outfile != nullptr)
            fclose(outfile);
        if (f != nullptr)
            fclose(f);
        if (context != nullptr){
            avcodec_free_context(&context);
        }
        if (decoded_frame != nullptr){
            av_frame_free(&decoded_frame);
        }
        if (pkt != nullptr){
            av_packet_free(&pkt);
        }
        if (swr_ctx != nullptr){
            swr_free(&swr_ctx);
        }
        return 0;
    } catch (std::exception& e) {
        on_error(env,callback,listener,CODEC_ERROR,"codec is not supported. exiting");
        if (outfile != nullptr)
            fclose(outfile);
        if (f != nullptr)
            fclose(f);
        if (context != nullptr){
            avcodec_free_context(&context);
        }
        if (decoded_frame != nullptr){
            av_frame_free(&decoded_frame);
        }
        if (pkt != nullptr){
            av_packet_free(&pkt);
        }
        if (swr_ctx != nullptr){
            swr_free(&swr_ctx);
        }
        return -1;
    }
}
```

其中：

##### Java调用C++函数

- 直接调用Java对应的native方法即可

##### C++调用Java类函数

通过jmethod、jclass进行调用

- 通过Java native函数传入的类在C++层均为jobject

- 通过`env->GetObjectClass(object)`得到object对应的class

- 通过`env->GetMethodID`方法得到想调用的方法名

  - 注意：`jmethodID GetMethodID(jclass clazz, const char* name, const char* sig)`最后一个为`sig`，需要描述函数的参数与返回值。

  - 格式为`([params;])L返回类型`

    - 基本类型

      | Java    | Native   | Signature |
      | ------- | -------- | --------- |
      | byte    | jbyte    | B         |
      | char    | jchar    | C         |
      | double  | jdouble  | D         |
      | float   | jfloat   | F         |
      | int     | jint     | I         |
      | short   | jshort   | S         |
      | long    | jlong    | J         |
      | boolean | jboolean | Z         |
      | void    | void     | V         |

    - 引用数据类型

      | Java      | Native        | Signature             |
      | --------- | ------------- | --------------------- |
      | 所有对象  | jobject       | L+classname +;        |
      | Class     | jclass        | Ljava/lang/Class;     |
      | String    | jstring       | Ljava/lang/String;    |
      | Throwable | jthrowable    | Ljava/lang/Throwable; |
      | Object[]  | jobjectArray  | [L+classname +;       |
      | byte[]    | jbyteArray    | [B                    |
      | char[]    | jcharArray    | [C                    |
      | double[]  | jdoubleArray  | [D                    |
      | float[]   | jfloatArray   | [F                    |
      | int[]     | jintArray     | [I                    |
      | short[]   | jshortArrsy   | [S                    |
      | long[]    | jlongArray    | [J                    |
      | boolean[] | jbooleanArray | [Z                    |

    - 一些例子![](https://img-blog.csdn.net/20170814173536226?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjcyNzg5NTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
    - 如果自己实在不知道怎么写，可以使用javac得到.java的class文件，再通过`javap -s -p`得到descriptor

- 通过call进行调用

  - `env->CallXXXMethod`调用，静态类/方法直接传入Class，非静态类/方法传入实现类（一般为Java层传入的jobject）

