---
layout: post
title: Github Action的使用
subtitle: Qt、CMake Project
author: Kingtous
date: 2020-12-20 13:11:27
header-img: img/about-bg-walle.jpg
catalog: True
tags:
- Android
---

此处记录使用Github Action编译Qt C++、CMake项目。

### 基本用法

```yaml
name: 任务名称
on : [触发event1、触发event2...] # event下还有很多配置参数如`paths-ignore`可以忽略相关文件如README
jobs: #job名称
	build:
		name: 任务名称
		runs-on: 系统名称
		env:
			xxx: xxx # 自行配置的环境变量
    steps:
    	- name: 第一步
    		id: 唯一id，可以通过id来得到某一步的输出以及属性
    		uses: 使用第三方的xxx插件
    		with:
    			...第三方插件的传入值
    	...
```

### CMake

安装LLVM 9打包`llvm`的一个简单项目

```yaml
name: CMake
# 在什么时候进行
on: [push]

env:
  # Customize the CMake build type here (Release, Debug, RelWithDebInfo, etc.)
  BUILD_TYPE: Release

jobs:
  build:
    # The CMake configure and build commands are platform agnostic and should work equally
    # well on Windows or Mac.  You can convert this to a matrix build if you need
    # cross-platform coverage.
    # See: https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/managing-complex-workflows#using-a-build-matrix
    runs-on: ubuntu-18.04

    steps:
    - uses: actions/checkout@v2
    
    - name: Installing LLVM 9
      run: sudo apt install -y llvm-9-dev
   
    - name: Installing libedit
      run: sudo apt install -y libedit-dev

    - name: Create Build Environment
      # Some projects don't allow in-source building, so create a separate build directory
      # We'll use this as our working directory for all subsequent commands
      run: cmake -E make_directory ${{runner.workspace}}/build

    - name: Configure CMake
      # Use a bash shell so we can use the same syntax for environment variable
      # access regardless of the host operating system
      shell: bash
      working-directory: ${{runner.workspace}}/build
      # Note the current convention is to use the -S and -B options here to specify source 
      # and build directories, but this is only available with CMake 3.13 and higher.  
      # The CMake binaries on the Github Actions machines are (as of this writing) 3.12
      run: cmake $GITHUB_WORKSPACE

    - name: Build
      working-directory: ${{runner.workspace}}/build
      shell: bash
      # Execute the build.  You can specify a specific target with "--target <NAME>"
      run: cmake --build .
    
    - name: Create Release
      id: create_release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.sha }}
        draft: false
        prerelease: false
    - name: Upload Release Asset
      id: upload-release-asset 
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }} # This pulls from the CREATE RELEASE step above, referencing it's ID to get its outputs object, which include a `upload_url`. See this blog post for more info: https://jasonet.co/posts/new-features-of-github-actions/#passing-data-to-future-steps 
        asset_path: ${{runner.workspace}}/build/llvm_kaleidoscope
        asset_name: llvm_kaleidoscope
        asset_content_type: application/x-executable
```

### Qt

在`MacOS`下使用macdeployqt打包，并且发布release的一个实例实例

```yaml
name: MacOS
on: 
  push:
    paths-ignore:
      - 'README.md'
      - 'LICENSE'
  pull_request:
    paths-ignore:
      - 'README.md'
      - 'LICENSE'
jobs:
  build:
    name: Build
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest]
        qt_ver: [5.15.2]
        qt_arch: [clang_64]
    env:
      targetName: NoteEasy
    steps:
      - name: cacheQt
        id: MacosCacheQt
        uses: actions/cache@v1 #一个缓存action，可以减少第二+次的重复下载
        with:
          path: ../Qt/${{matrix.qt_ver}}/${{matrix.qt_arch}}
          key: ${{ runner.os }}-Qt/${{matrix.qt_ver}}/${{matrix.qt_arch}}  
      - name: Install Qt
        # if: steps.MacosCacheQt.outputs.cache-hit != 'true'
        uses: jurplel/install-qt-action@v2
        with:
          version: ${{ matrix.qt_ver }}
          cached: ${{ steps.MacosCacheQt.outputs.cache-hit }}
      - uses: actions/checkout@v1
        with:
          fetch-depth: 1
      - name: build macos
        run: |
          qmake
          make
      # tag 打包
      - name: package
        if: startsWith(github.event.ref, 'refs/tags/')
        run: |
          # 拷贝依赖
          macdeployqt bin/${targetName}.app -qmldir=. -verbose=1 -dmg
      # tag 查询github-Release
      - name: queryRelease
        id: queryReleaseMacos
        if: startsWith(github.event.ref, 'refs/tags/')
        shell: pwsh
        env:
          githubFullName: ${{ github.event.repository.full_name }}
          ref: ${{ github.event.ref }}
        run: |
          [string]$tag = ${env:ref}.Substring(${env:ref}.LastIndexOf('/') + 1)
          [string]$url = 'https://api.github.com/repos/' + ${env:githubFullName} + '/releases/tags/' + ${tag}
          $response={}
          try {
            $response = Invoke-RestMethod -Uri $url -Method Get
          } catch {
            Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__ 
            Write-Host "StatusDescription:" $_.Exception.Response.StatusDescription
            # 没查到，输出
            echo "::set-output name=needCreateRelease::true"  
            return
          }
          [string]$latestUpUrl = $response.upload_url
          Write-Host 'latestUpUrl:'$latestUpUrl
          if ($latestUpUrl.Length -eq 0) {
            # 没查到，输出
            echo "::set-output name=needCreateRelease::true"  
          }
      # tag 创建github-Release
      - name: createReleaseWin
        id: createReleaseWin
        if: startsWith(github.event.ref, 'refs/tags/') && steps.queryReleaseMacos.outputs.needCreateRelease == 'true'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        uses: actions/create-release@v1.0.0
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: ${{ github.event.head_commit.message }}
          draft: false
          prerelease: false
      # 重定向upload_url到环境变量uploadUrl。
      - name: getLatestTagRelease
        # tag 上一步无论成功还是失败都执行
        if: startsWith(github.event.ref, 'refs/tags/')
        shell: pwsh
        env:
          githubFullName: ${{ github.event.repository.full_name }}
          upUrl: ${{ steps.queryReleaseMacos.outputs.upload_url }}
          ref: ${{ github.event.ref }}
        run: |
          # upUrl不为空，导出就完事
          if (${env:upUrl}.Length -gt 0) {
              $v=${env:upUrl}
              echo "::set-env name=uploadUrl::$v"
              return
          } 
          [string]$tag = ${env:ref}.Substring(${env:ref}.LastIndexOf('/') + 1)
          [string]$url = 'https://api.github.com/repos/' + ${env:githubFullName} + '/releases/tags/' + ${tag}
          $response = Invoke-RestMethod -Uri $url -Method Get
          [string]$latestUpUrl = $response.upload_url
          Write-Host 'latestUpUrl:'$latestUpUrl
          echo "::set-env name=uploadUrl::$latestUpUrl"
          Write-Host 'env uploadUrl:'${env:uploadUrl}
      # tag 上传Release
      - name: uploadRelease
        id: uploadRelease
        if: startsWith(github.event.ref, 'refs/tags/')
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        uses: actions/upload-release-asset@v1.0.1
        with:
          upload_url: ${{ env.uploadUrl }}
          asset_path: ./bin/${{ env.targetName }}.dmg
          asset_name: ${{ env.targetName }}.dmg
          asset_content_type: application/applefile
```

