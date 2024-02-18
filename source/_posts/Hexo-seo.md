---
title: Hexo-seo Optimization Tips
date: 2021-02-06 20:37:12
tags: 
	- git
	- Web deployment
	- reference
id: Hexo-seo
---

Usually Baidu, Google and other search engines actively included in our site is not very likely. This requires us to SEO (Search Engine Optimization) the site, and for blog sites built on Hexo, the Hexo-SEO optimization strategy is used here.

<!-- more -->

# 1. Generate the sitemap files

Two Hexo plug-ins need to be installed first:

```shell
npm install hexo-generator-sitemap --save		
npm install hexo-generator-baidu-sitemap --save
```

Open the configuration file `_config.yml` in the root tree and add:

```yaml
sitemap:
	path: sitemap.xml
baidusitemap:
	path: baidusitemap.xml
```

Restart Hexo again and access [localhost:4000/sitemap.xml](localhost:4000/sitemap.xml) and [localhost:4000/baidusitemap.xml](localhost:4000/baidusitemap.xml) locally to check the two sitemap files. Be sure to ensure that the sitemap file is displayed properly in your browser and that no errors are reported. Possible causes of errors are the following:

- The `sitemap.xml` file is in the wrong encoding format, you need to use an editor to convert the encoding format to `UTF-8` or `UNICODE`.
- The URL saved in `sitemap.xml` does not contain `&`, i.e. the path to the article in Hexo does not contain `&`. If it does, you need to replace `&` in the URL with `&amp;`.

There may also be other errors, which can be located by the error message and are not repeated here.

# 2. Push to Google and Baidu

## (1) Baidu → [Add personal website](https://ziyuan.baidu.com/site/siteadd?siteurl=)

> The add file approach is not feasible, hexo will handle html files. So the choice is, add html tags to `head.ejs`

[Baidu Dashboard](https://ziyuan.baidu.com/dashboard/index)

1.1 Manually submit [baidusitemap.xml](https://ziyuan.baidu.com/linksubmit/index) (There is also an automatic submission code inside)

1.2 You can use "Crawl Diagnosis", manually - Baidu crawl

1.3 Robots → detect and update

> ##### Robots configuration
>
> ```yaml
> User-agent: *
> Allow: /
> Allow: /home/
> Allow: /archives/
> Allow: /about/
> Disallow: /vendors/
> Disallow: /js/
> Disallow: /css/
> Disallow: /fonts/
> Disallow: /vendors/
> Disallow: /fancybox/
> 
> Sitemap: http://yoursite/sitemap.xml
> Sitemap: http://yoursite/baidusitemap.xml
> ```
>
> `Allow` means allowed to be accessed, `Disallow` means not allowed. Note that the last two sitemaps are URL to sitemaps. `Robots.txt` files need to be placed in the root of the page, in line with the `sitemap` file. In the Hexo blog, it needs to be placed in `./public`

## (2)Google → [Add personal website](https://search.google.com/search-console/welcome)

Similar to Baidu, it also adds html tags to `head.ejs`.

**You can also verify websites on baidu and google by means of DNS, here I Google is through the DNS method. See the official guide for details, so I won't go into it again here.**

##### Manual sitemap submission, even for individual sites: 

- [GoogleSearchConsole](https://search.google.com/search-console) → sitemap → Enter the corresponding URL for sitemap.xml → Submit

Verification is good, after two days or so Baidu and Google will be able to include your site

Test method: (respectively in google and baidu search their site or site url title content)

# 3. Remove dead links regularly

[https://www.google.com/webmasters/tools/removals](https://www.google.com/webmasters/tools/removals)