---
layout: post
status: publish
published: true
title: OpenMP(一)：使用OpenMP开发高效率多文件夹大小、数量统计程序
author: Kingtous
author_login: jintao
author_email: me@kingtous.cn
wordpress_id: 300
wordpress_url: https://hq-note.kingtous.cn/?p=300
date: '2021-09-28 21:42:57 +0800'
date_gmt: '2021-09-28 13:42:57 +0800'
categories:
- OpenMP
tags: []
---
<p><!-- wp:paragraph --></p>
<p>任务：递归统计一个文件夹总大小，以及所有文件数量（只包含regular file）</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>文件大小&mdash;&mdash;POSIX</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>在POSIX中，定义一个文件、文件夹的信息使用struct stat，如下：</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:enlighter/codeblock {"language":"cpp","theme":"monokai","linenumbers":"true"} --></p>
<pre class="EnlighterJSRAW" data-enlighter-language="cpp" data-enlighter-theme="monokai" data-enlighter-highlight="" data-enlighter-linenumbers="true" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">struct stat {
    dev_t     st_dev;     /* ID of device containing file */          //  文件的设备编号
    ino_t     st_ino;     /* inode number */                               //  结点
    mode_t    st_mode;    /* protection */                               //  文件的类型和存取的权限
    nlink_t   st_nlink;   /* number of hard links */                   // 连到该文件的硬链接数目，新建的文件则硬连接数为 1
    uid_t     st_uid;     /* user ID of owner */                          //    用户ID
    gid_t     st_gid;     /* group ID of owner */                        // 组ID
    dev_t     st_rdev;    /* device ID (if special file) */            // 若此文件为设备文件，则为其设备的编号
    off_t     st_size;    /* total size, in bytes */                        //  文件字节数(文件大小)
    blksize_t st_blksize; /* blocksize for filesystem I/O */       // 块大小
    blkcnt_t  st_blocks;  /* number of 512B blocks allocated */           // 块数
    time_t    st_atime;   /* time of last access */                                  //  最后一次访问时间
    time_t    st_mtime;   /* time of last modification */                         // 最后一次修改时间
    time_t    st_ctime;   /* time of last status change */                       // 最后一次改变时间
};</pre>
<p><!-- /wp:enlighter/codeblock --></p>
<p><!-- wp:paragraph --></p>
<p>此处我们使用到st_mode获取文件类型，st_size获取文件字节数大小（bytes）。</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>需要&amp;个S_IFMT（文件类型掩码）才能进行判断，如：st_mode &amp; S_IFMT == S_IFDIR判断是否是文件夹，S_IFREG（regular普通文件）判断是否是文件等</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:heading --></p>
<h2>统计时间</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>使用chrono库实现毫秒级统计</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:enlighter/codeblock --></p>
<p><!-- /wp:enlighter/codeblock --></p>
<p><!-- wp:heading --></p>
<h2>文件夹遍历</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>使用dirent.h提供的`DIR`数据结构进行判断</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>opendir返回一个可访问的DIR指针</li>
<li>每次对DIR调用readdir，会依次返回下一个文件的dirent指针</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:heading --></p>
<h2>OpenMP模型</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>宏定义：</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>pragma omp parallel
<ul>
<li>定义一个并行域，是创建OpenMP域的必备宏</li>
<li>firstprivate相当于同名赋值、private相当于不传值</li>
</ul>
</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:list --></p>
<ul>
<li>pragma omp task
<ul>
<li>定义一个OpenMP task，声明一个任务</li>
<li>untied表示，该task内的代码可以由多个线程执行，否则<strong>整个</strong>task只能由一个线程完成</li>
</ul>
</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:quote --></p>
<blockquote class="wp-block-quote"><p>The code associated with a task construct will be executed only once.</p>
<p>A task is <strong>tied</strong> if the code is executed by the same thread from</p>
<p>beginning to end. Otherwise, the task is untied (the code can be</p>
<p>executed by more than one thread).</p>
<p><cite><a href="http://icl.cs.utk.edu/classes/cosc462/2016/pdf/W45L1%20-%20OpenMP%20Tasks.pdf">Lecture 6 - OpenMP Tasks (utk.edu)</a></cite></p></blockquote>
<p><!-- /wp:quote --></p>
<p><!-- wp:image {"align":"center","id":310,"sizeSlug":"large","linkDestination":"none"} --></p>
<div class="wp-block-image">
<figure class="aligncenter size-large"><img src="https://hq-note.kingtous.cn/wp-content/uploads/2021/09/image-23-1024x568.png" alt="" class="wp-image-310"/></figure>
</div>
<p><!-- /wp:image --></p>
<p><!-- wp:list --></p>
<ul>
<li>pragma omp taskwait
<ul>
<li>等待当前parallel中的所有task执行完后继续执行</li>
</ul>
</li>
<li>pragma omp single
<ul>
<li>single下的代码域由单个线程执行（parallel默认会调用<strong>最大系统推荐的线程数</strong>来执行代码域，有时并行域中只需要执行一次）</li>
</ul>
</li>
<li>pragma omp critical
<ul>
<li>表示这段代码一次只能一个进程执行</li>
</ul>
</li>
<li>pragma omp atomic
<ul>
<li>表示这段代码一次只能一个进程执行，与Critical的区别？
<ul>
<li>atomic可以借助硬件实现，开销较小</li>
</ul>
</li>
</ul>
</li>
<li>pragma omp section与task的区别
<ul>
<li><a href="https://stackoverflow.com/questions/13788638/difference-between-section-and-task-openmp">c - Difference between section and task openmp - Stack Overflow</a></li>
</ul>
</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:paragraph --></p>
<p>An OpenMP critical section is completely general - it can surround any arbitrary block of code. You pay for that generality, however, by incurring significant overhead every time a thread enters and exits the critical section (on top of the inherent cost of serialization).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>(In addition, in OpenMP all unnamed critical sections are considered identical (if you prefer, there's only one lock for all unnamed critical sections), so that if one thread is in one [unnamed] critical section as above, no thread can enter any [unnamed] critical section. As you might guess, you can get around this by using named critical sections).</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>An atomic operation has much lower overhead. Where available, it takes advantage on the hardware providing (say) an atomic increment operation; in that case there's no lock/unlock needed on entering/exiting the line of code, it just does the atomic increment which the hardware tells you can't be interfered with.</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>函数：</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:list --></p>
<ul>
<li>omp_get_num_threads...</li>
</ul>
<p><!-- /wp:list --></p>
<p><!-- wp:paragraph --></p>
<p>后面慢慢补充~</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:heading --></p>
<h2>程序代码</h2>
<p><!-- /wp:heading --></p>
<p><!-- wp:paragraph --></p>
<p>使用C++14标准编译</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:enlighter/codeblock {"language":"cpp","theme":"monokai","linenumbers":"true"} --></p>

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <cstring>
#include <iostream>
#include <dirent.h>
#include <vector>
#include <omp.h>
#include <set>
#include <chrono>

using std::chrono::duration_cast;
using std::chrono::milliseconds;
using std::chrono::seconds;
using std::chrono::system_clock;

using namespace std;

class FileStater
{
public:
    uint64_t calculate()
    {
        auto res = __stat(this->basePath);
#pragma omp taskwait
        return res;
    }

    uint64_t __stat(const string &basePath)
    {
        uint64_t totalSizeCount = 0;
        struct stat p;
        auto ret = stat(this->basePath.c_str(), &p);
        if (ret == -1)
        {
            printf("warning: open path %s failed.\n", basePath.c_str());
            return 0;
        }
        if ((p.st_mode & S_IFMT) == S_IFDIR)
        {
            // 是文件夹
            DIR *dirp = opendir(basePath.c_str());
            if (dirp == NULL)
            {
                printf("warning: open directory %s failed.\n", basePath.c_str());
                return 0;
            }
            dirent *dir = NULL;
            while ((dir = readdir(dirp)) != NULL)
            {
                if (dir->d_type == DT_DIR)
                {
                    if (strcmp(dir->d_name, ".") != 0 && strcmp(dir->d_name, "..") != 0)
                    {
#pragma omp parallel firstprivate(dir, basePath)
#pragma omp single
#pragma omp task untied
                        // 开始stat
                        {
                            auto subSize = __stat(basePath + "/" + dir->d_name);
#pragma omp critical
                            {
                                totalSizeCount += subSize;
                                this->fileCounts++;
                            }
                        }
                    }
                }
                else

                {
                    struct stat filep;
                    auto ret = stat((basePath + "/" + dir->d_name).c_str(), &filep);
                    if (ret == -1)
                    {
                        printf("warning: stat file %s failed.\n", (basePath + "/" + dir->d_name).c_str());
                    }
                    else
                    {
#pragma omp critical
                        {
                            totalSizeCount += filep.st_size;
                            this->fileCounts++;
                        }
                    }
                }
            }
            closedir(dirp);
            // 获取所有文件，每个多线程执行__stat
        }
        else if ((p.st_mode & S_IFMT) == S_IFREG)
        {
            // 是普通文件
            // 直接计算文件大小
            totalSizeCount += p.st_size;
        }
        return totalSizeCount;
    }

    FileStater(const string &basePath)
    {
        this->basePath = std::move(basePath);
    }

    int fileCounts{0};

private:
    string basePath;
};

int main(int argc, char const *argv[])
{
    auto t1 = duration_cast<milliseconds>(system_clock::now().time_since_epoch()).count();

#pragma omp parallel for
    for (int i = 1; i < argc; ++i)
    {
        printf("start stat in thread %d\n", omp_get_thread_num());
        auto fs = new FileStater{argv[i]};
        auto folderSize = fs->calculate() / 1024 / 1024;
        printf("info for:%s\n",argv[i]);
        printf("folder size: %lld MB\n", folderSize);
        printf("file count: %d MB\n", fs->fileCounts);
    }
    auto t2 = duration_cast<milliseconds>(system_clock::now().time_since_epoch()).count();
    cout << "time cost: " << (t2 - t1) << "ms" << endl;
    return 0;
}

```

<p><!-- /wp:enlighter/codeblock --></p>
