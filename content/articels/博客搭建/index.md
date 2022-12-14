---
title: "博客搭建简述"
showSummary: true
summary: "简述一下博客搭建过程。<br/>框架使用的是用GO语言编写的Hugo，Hugo模板用的是blowfish。"
layoutBackgroundBlur: true
layoutBackgroundHeaderSpace: false
date: 2022-12-09
---


## hugo搭建
&emsp;&emsp;具体文件详见 [HUGO安装文档](https://gohugo.io/installation/)，下面主要解释一下docker部署的方式。
{{< alert "circle-info">}}
**docker镜像地址**：https://hub.docker.com/r/klakegg/hugo
<br/>
**github地址**：https://github.com/klakegg/docker-hugo
{{< /alert >}}


&emsp;&emsp;lakegg/hugo镜像有基于[Busybox](https://busybox.net/)、[Alpine](https://www.alpinelinux.org/)、[Debian](https://www.debian.org/) 和
[Ubuntu](https://ubuntu.com/)4个系统.其中每个系统又分为normal、ext、onbuild、ci、ext-ci、ext-onbuild 6种，Busybox没有ext相关，下面简要介绍一下几种的区别

* **normal**：只有基础的hugo命令。默认Entrypoint是hugo，所以common只需要写server，即可执行hogu server
* **ext**：拓展版，在normal基础上添加了go、git、nodejs等拓展工具 (推荐)
* **ci**: 主要用于持续集成/部署。在normal基础上，新增了环境变量HUGO_ENV，去掉了默认Entrypoint，需要自行添加容器运行命令
* **onbuild**: 此版本主要用于编译构建并可以分开源码和产出。默认源码文件夹是/src，产出文件夹是.target，可以通过HUGO_DIR和HUGO_DESTINATION_ARG进行自定义设置。

docker镜像版本如下：
<table>
    <thead>
        <tr>
            <th ></th>
            <th >busybox</th>
            <th >Alpine</th>
            <th >Alpine with Asciidoctor</th>
            <th >Alpine with Pandoc</th>
            <th >Debian</th>
            <th >Ubuntu</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td style="font-weight: bold; border-top-width: 1.5px; ">normal</td>
            <td style="border-top-width: 1.5px;">latest<br/>busybox</td>
            <td style="border-top-width: 1.5px;">alpine</td>
            <td style="border-top-width: 1.5px;">asciidoctor</td>
            <td style="border-top-width: 1.5px;">pandoc</td>
            <td style="border-top-width: 1.5px;">debian</td>
            <td style="border-top-width: 1.5px;">ubuntu</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td style="font-weight: bold; border-top-width: 1.5px; border-top-color: snow">ext</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">-</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-alpine</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-asciidoctor</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-pandoc</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-debian</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-ubuntu</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td style="font-weight: bold;border-top-width: 1.5px; border-top-color: snow">ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ci<br/>busybox-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">alpine-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">asciidoctor-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">pandoc-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">debian-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ubuntu-ci</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td style="font-weight: bold;border-top-width: 1.5px; border-top-color: snow">onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">onbuild<br/>busybox-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">alpine-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">asciidoctor-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-pandoc-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">debian-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ubuntu-onbuild</td>
        </tr>
    </tbody>
     <tbody>
        <tr>
            <td style="font-weight: bold;border-top-width: 1.5px; border-top-color: snow">ext-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">-</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-alpine-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-asciidoctor-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-pandoc-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-debian-ci</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-ubuntu-ci</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td style="font-weight: bold;border-top-width: 1.5px; border-top-color: snow">ext-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">-</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-alpine-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-asciidoctor-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-pandoc-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-debian-onbuild</td>
            <td style="border-top-width: 1.5px; border-top-color: snow">ext-ubuntu-onbuild</td>
        </tr>
    </tbody>
</table>

> hugo模板可以参考 [https://themes.gohugo.io/](https://themes.gohugo.io/)
 
## Blowfish模板使用

&emsp;&emsp;Hugo的主题主要是用了blowfish，下面主要对blowfish的使用展开简要说明
{{< alert "circle-info">}}
[Hugo theme - blowfish](https://themes.gohugo.io/themes/blowfish)<br/>
[blowfish github地址](https://github.com/nunocoracao/blowfish)
{{< /alert >}}

&emsp;&emsp;blowfish的github主要介绍了使用git submodules和Hugo的部署方式，这里主要介绍一下如何使用上面介绍的hugo docker镜像进行部署

1. go.mod中添加blowfish
```
require github.com/nunocoracao/blowfish/v2 version // indirect
```
2. config/_default/module.toml文件添加：
```toml
[[imports]]
path = "github.com/nunocoracao/blowfish/v2"
```
3. 将根目录引用到镜像内的/src目录下，启动docker即可


### 自定义浏览器角标
&emsp;&emsp;在[favicon.io](https://favicon.io/)讲自己的图片生成为各种尺寸的icon，直接解压在favicon.io下载好的icon压缩包，并放在/static目录下即可

### 自定义ICON
&emsp;&emsp;将自定义的svg文件放在/asserts/icons目录下，为了使ICON和主题自适应，需要在svg文件中添加属性 fill="currentColor" 如下：
```svg
<svg>
    <<path fill="currentColor" d="xxx"/>
</svg>
```

### blowfish文档
&emsp;&emsp;使用blowfish搭建博客，具体参考[文档](https://nunocoracao.github.io/blowfish/docs/)
