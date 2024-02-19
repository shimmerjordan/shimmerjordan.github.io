---
title: Free CDN for WebPage
date: 2021-01-27 13:48
tags: 
	- Web deployment
	- git
categories: 新手指北
id: freeWebpageCDN
---

The previous article {% post_link  FreeStaticWebpageDeployment  Free Static Webpage Deployment %}  has explained in full how to deploy a free static website, here are further instructions on how to get free CDN services to improve the speed of access to your website in different parts of the world.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/02/06/yYsRE9.jpg" width=35%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">CDN</div>
</center>

<!-- more -->

# Overview of steps

- Ensure deployment of mid-owned cloud repository is completed;
- Login to Vercel;
- Import the corresponding cloud repository containing the pages to be deployed;
- [Option] Custom Domains;
- Later remarks.

# Ensure deployment of mid-owned cloud repository is completed

Here you need to make sure that you have web resources that can be accessed properly, as can be seen in {% post_link  FreeStaticWebpageDeployment  Free Static Webpage Deployment %}. Of course, if you are considering using vercel acceleration here, the cloud repository name does not need to be specified as in the previous article, it can be completely customised. This is because vercel can copy and host the cloud repository.

# Login to Vercel

You can log in to Vercel using github, gitLab and bitbucket, and you can bring in repositories from other platforms (if you are authorised to do so), regardless of which account you are registered under.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/27/sxDG38.png" width=70%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Sign Up</div>
</center>

# Import the corresponding cloud repository containing the pages to be deployed

You can follow the guidelines for importing repositories or just click on import third party repositories. By doing this it is clear why you don't need a specific named cloud repository, the principle of vercel is to automatically pull the bound cloud repository and deploy it in vercel. The site to be deployed can therefore not rely on a GithubPage.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/27/sxD44x.png" width=60%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Import Git Respository</div>
</center>



It is important to note here that if the cloud repository is simply a static web page that can be accessed directly without additional deployment operations via index.html etc. Here the following options need to be set at the time of introduction, the main points to note are the following:

- Ensure that `FRAMEWORK PRESET` is `OTHER`, i.e. no additional deployment and installation is required;

- Check the `OVERRIDE` option for `BUILD COMMAND`, `OUTPUT DICTIONARY`,`INSTALL COMMAND`,`DEVELOPMENT COMMAND` and change its content to empty. (Even though all content is removed and no input is made, the empty form here will still show some content in grey, i.e. the text will be disabled, as shown below).

  <center>
      <img style="border-radius: 0.3125em;
      box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
      src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/27/sxrYPx.png" width=60%>
      <br>
      <div style="color:orange; border-bottom: 1px solid #d9d9d9;
      display: inline-block;
      color: #999;
      padding: 2px;">OVERRIDE COMMAND</div>
  </center>

By this point it can be accessed using the second level domain name provided by vercel. It is also possible to add this second level domain to the Custom domain of the GithubPage to improve access speed. If domain customisation is required, then the following deployment can be done.

# [Option] Custom Domains

Select your project card to enter the project, click Settings-Domains and enter your domain name Add to add:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/27/sxyejU.png" width=65%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Add domains</div>
</center>

Then resolve the domain name.

- Root domain recommended: Record type: A  Record value: `76.76.21.21`
- Sub-domain recommended: Record type: CNAME  Record value: `cname.vercel-dns.com`

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/01/27/sxcTnU.png" width=65%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Resolve the domain name</div>
</center>

It will automatically validate the domain name, enable ssl, redirect and all that is supported, and you can see the record you have added by editing it yourself.

# Later remarks

Vercel automatically pulls updates from the introduced cloud repositories, so updates only need to be pushed to the corresponding d repository

This is all I have done, and here are the current pages I have successfully deployed by this method:

- [My blog (github)](https://blog.shimmerjordan.eu.org/)

- [My CV (bitbucket)](https://cv.shimmerjordan.eu.org/)


Similar CDN acceleration services include [jsDelivr](https://www.jsdelivr.com/), and here I have chosen to use [jsDelivr](https://www.jsdelivr.com/) in conjunction with github as my personal image bed.

Its use is shown below:

>  https://gcore.jsdelivr.net/gh/username/repositoryName@branch/filePath
>
> - gh: Indicates this is a github resource;
> - username: your username of github;
> - repositoryName: Name of the repository corresponding to the resource;
> - branch: The branch of the repository to which the resource corresponds, usually master, or the version number of the release;
> - filePath: Path to the file.
> - **For more information, please refer to [https://github.com/jsdelivr/data.jsdelivr.com](https://github.com/jsdelivr/data.jsdelivr.com)**

Use the following image as an example (723KB):

- Original link: [origin github link (https://github.com/shimmerjordan/shimmerjordan.github.io/blob/master/images/cover5.jpg)](https://github.com/shimmerjordan/shimmerjordan.github.io/blob/master/images/cover5.jpg)

- Preview: 

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://github.com/shimmerjordan/shimmerjordan.github.io/blob/master/images/cover5.jpg?raw=true" width=65%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Original link</div>
</center>

- Link to the accelerated resources referenced via jsDelivr: [CDN link (https://gcore.jsdelivr.net/gh/shimmerjordan/shimmerjordan.github.io@master/images/cover5.jpg)](https://gcore.jsdelivr.net/gh/shimmerjordan/shimmerjordan.github.io@master/images/cover5.jpg)
- Preview:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/shimmerjordan.github.io@master/images/cover5.jpg" width=65%>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">CDN link</div>
</center>

Of course, it is also possible to operate directly on github resources via tools such as PicGo and PicX, eliminating the need for troublesome git operations. This requires setting up read and write access to the repo via [https://github.com/settings/tokens](https://github.com/settings/tokens) , and generating the appropriate token, which is not covered here.

# TIPS

> Jsdelivr国内的CDN服务被DNS污染了，被指向了Google、Twitter和 Facebook 的 IP 地址，导致使用CDN服务加速的链接访问失败。

Jsdelivr国内的CDN服务被DNS污染，往往一般是cdn.jsdelivr.net被DNS污染了，而其他代替的地址没有被污染，比如fastly.jsdelivr.net、gcore.jsdelivr.net等。这时候我们就可以批量把图片或者其他静态资源链接中的gcore.jsdelivr.net替换为别的可用的地址（下面自己选一个可用的），等官方修复回去后再替换回去就行了。

- 无CDN加速`https://raw.githubusercontent.com/shimmerjordan/shimmerjordan.github.io/master/cover5.jpg`
- Jsdelivr的DNS被污染`https://cdn.jsdelivr.net/gh/shimmerjordan/shimmerjordan.github.io@master/images/cover5.jpg`
- Jsdelivr替换后的`https://gcore.jsdelivr.net/gh/shimmerjordan/shimmerjordan.github.io@master/images/cover5.jpg`

## 可替换的选项

- fastly.jsdelivr.net
- gcore.jsdelivr.net
- testingcf.jsdelivr.net
- test1.jsdelivr.net