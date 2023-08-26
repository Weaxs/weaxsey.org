---
title: "Hugo + umami 博客统计面板"
showSummary: true
summary: "使用 umami 为 Hugo 博客添加统计面板"
date: 2023-08-25
tags: ["编程"]
---


## 引言

umami是一个用于网页访问统计的工具，是 Google Analytics的替代方案。本文主要介绍使用 umami 为博客添加访客统计面板

{{< github repo="umami-software/umami" >}}


我们先介绍以下以下几种方案：

| 平台 | 部署管理方式 | 优点                       | 缺点 | 网站 |
| --- | --- |--------------------------| --- | --- |
| umami cloud | 平台管理 | 免费、快捷、简单                 | 免费的仅保留1年数据，仅3个网站 | https://cloud.umami.is/ |
| Netlify + Neon | 自运维部署。Neon部署PostgreSQL，Netlify部署umami。  | 免费，数据可以永久保留，统计数据无限制      | 部署麻烦，需要2个平台，主要因为Netlify不能部署DB | https://www.netlify.com/  https://neon.tech/ |
| Railway | 自运维部署。Railway上部署umami和PostgreSQL。 | 部署相对方便，自运维数据永久保留，统计数据无限制 | 不免费，最少$5/month | https://railway.app/ |

> ✨ 具体的计费方式、规格等问题可以查看平台的Pricing
>

## Umami Cloud

### 配置

去 [https://cloud.umami.is/](https://cloud.umami.is/) 进行注册，添加需要统计的domain

### Tracking Code

进入 [https://cloud.umami.is/settings/websites](https://cloud.umami.is/settings/websites)，对需要查看的网站点击编辑，可以找到我们需要在网站上添加的 `Tracking code`

![TrackingCode.png](TrackingCode.png))

## **Netlify + Neon**

### 部署

1. 去 Neon 平台创建 PostgreSQL，创建成功后记录DATABASE_URL，例如：`postgres://user:passwd@endpoint-pooler.region.aws.neon.build/neondb`
2. 去 Netlify 平台部署 umami 服务
    1. fork umami 的 github 源码
    2. 在 Netlify 平台，对fork的代码库进行构建
    3. 添加环境变量 DATABASE_URL，把从Neon平台拿到的数据库地址作为value
    4. 服务部署
    5. 更换成自己的域名地址，也可以用平台提供的

### Tracking Code

部署成功之后，通过平台提供的域名登录 umami 控制台，默认的用户名是admin，密码是umami。

在「设置」- 「网站」 - 「编辑」中，可以找到我们需要的 `Tracking Code`

![TrackingCode2.png](TrackingCode2.png)

### 参考

[https://umami.is/docs/running-on-neon](https://umami.is/docs/running-on-neon)

[https://umami.is/docs/running-on-netlify](https://umami.is/docs/running-on-netlify)

## **Railway**

### 部署

1. 创建 **Project**
2. 在 **Project** 中添加 **PostgreSQL**
3. 在 Project 中添加并部署 umami 服务
    1. 选择 fork 后的 umami 代码库
    2. 添加环境变量`DATABASE = ${{Postgres.DATABASE_URL}}`
    3. 服务部署
    4. 可以设置自定义域名，也可以使用平台提供的

![Railway.png](Railway.png)

### Tracking Code

自运维的 umami 服务，查看 Tracking Code 的地方都一样，如下图：

![TrackingCode2.png](TrackingCode2.png)

### 参考

[https://umami.is/docs/running-on-railway](https://umami.is/docs/running-on-railway)

## 在 Hugo 框架下配置

通过对 umami 的熟悉，我们可以知道，想要对网站添加 umami tracking，需要在 `<head>` 标签中添加`<script async defer src="http://mywebsite.com/umami.js" data-website-id="94db1cb1-74f4-4a40-ad6c-962362670409"></script>`

那么在 Hugo 框架下，如何在 `<head>` 中添加呢？

下面我们需要了解Hugo 中的 [Partial](https://gohugo.io/templates/partials/)。它是用于页面模板中的上下文组件的，例如统一配置网站中网页的header 和 footer，具体的配置目录是`layouts/partials/*<PARTIALNAME>.html`

回到问题，我们需要在 `<head>` 中添加 umami script，其实就意味着我们需要添加在 `layouts/partials/head.html` 中，那么此时有两种方式：

1. 直接写在 `head.html` 中
2. 写在 partial 的别的目录中，并在 `head.html`中通过 **`{{** partial "<PATH>/<PARTIAL>.html" . **}}**` 的方式引入进来

### Ladder 主题

ladder 主题的代码库是

{{< github repo="guangzhengli/hugo-theme-ladder" >}}

从源码中的`layouts/partials/common/analytics.html` 可以看到这个主题对 umami 的引入

```html
{{ template "_internal/google_analytics.html" . }}

{{ if .Site.Params.analytics.umami.enable }}
<script async defer data-website-id="{{ .Site.Params.analytics.umami.website_id }}" src="{{ .Site.Params.analytics.umami.url }}"></script>
{{ end }}
```

同时我们可以看到源码`layouts/partials/head.html` 中，将 umami 的 `js` 引入到了 head 中，具体如下：

```html
{{ partial "common/analytics.html" . }}
```

综上，如果我们是基于 Ladder 搭建的博客，那么我们直接在配置文件中添加即可。具体是在项目根目录中的 `config.yml` 中添加如下配置：

```yaml
params:
	analytics:
	  umami:
	    enable: true
        website_id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
        url: https://analytics.umami.is/script.js
```

### Blowfish 主题

Blowfish 主题的代码库是

{{< github repo="nunocoracao/blowfish" >}}


Blowfish 支持的 analytics 目录在`layouts/partials/analytics`，在这里我们可以看到，目前Blowfish 只支持一下两种 analytics:

1. Google Analytics
2. Fathom Analytics

在 `layouts/partials/head.html` 中我们可以看到，Blowfish 这里将 analytics 目录下的 `main.html` 引入到了 `head.html` 中

```html
  {{/* Analytics */}}
  {{ partial "analytics/main.html" .Site }}
  {{/* Extend head - eg. for custom analytics scripts, etc. */}}
  {{ if templates.Exists "partials/extend-head.html" }}
  {{ partialCached "extend-head.html" .Site }}
  {{ end }}
```

下面我们提供两种添加 umami analytics 的方式

1. 不修改源码版本
    1. 添加 `layouts/partials/extend-head.html` 文件，引入 umami script
    2. 在 `config/_default/params.toml` 中添加参数
2. 修改源码版本
    1. 添加`layouts/partials/analytics/umami.html` ，具体代码同 方法一中的extend-head.html
    2. 修改 `layouts/partials/analytics/main.html`，通过  `{{ partial "analytics/umami.html" .Site }}` 引入 umami.html
    3. 在 `config/_default/params.toml` 中添加参数

下面给出示例版本，html 文件引入如下：

```html
{{ if isset site.Params.umamiAnalytics "domain" }}
<script async src="https://{{ site.Params.umamiAnalytics.domain }}/script.js" data-website-id="{{ site.Params.umamiAnalytics.websiteid }}"></script>
{{ else }}
<script async src="https://analytics.umami.is/script.js" data-website-id="{{ site.Params.umamiAnalytics.websiteid }}"></script>
{{ end }}
```

在 `config/_default/params.toml` 中添加参数

```toml
[umamiAnalytics]
  websiteid = "xxxxx"
  domain = "umami.example.com"
```

## umami Tracker 参数

我们上面给到的 script 脚本，只用到了 `src` 和 `data-website-id` 参数，还可以添加一些别的参数，具体的文档在 [tracker-configuration](https://umami.is/docs/tracker-configuration)，下面我们做个总结

| 参数 | 用途 |
| --- | --- |
| data-host-url | 默认情况下，trace信息默认是发送到 js 部署的host，即src中的 host，可以通过此配置修改发送到的 host |
| data-auto-track | 默认情况下为true，自动 trace 所有的页面访问和事件，可以将其设为false，并配合 tracker functions 来设置事件和访问行为。 |
| data-do-not-track | 默认情况下为false，如果设置为true代表尊重访问者的“请勿跟踪”设置 |
| data-cache | 默认情况下为false，如果某个用户有大量访问，设置为true可以通过缓存提高用户的访问性能。 |
| data-domains | 如果你仅仅想在特定域名上trace，可以通过添加此参数来限制。 |