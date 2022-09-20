---
title: "How to Build Blog With Hugo on Github"
date: "2018-10-26"
tags: ["hugo"]
categories: ["hugo"]
author: "steve dong"
draft: false
reward: true
---



### Hugo on your MacOS

 * Setting github

 1. create a repo named blogs for store hugo programes.
 2. Create another repo named echowings.github.io to host websit.
 3. generate ssh key and regist it on github settings. 

 * Install hugo

 ```bash
  
brew install hugo


```

 * Create a blog site.

  ``` bash
  
hugo new site blogs

```

 * init git

 ``` bash
 
cd blogs
git init
#Add remote repo. 
git remote add origin git@github.com:echowings/blogs.git
#Add a theme for your hugo.
git submodule add -b master https://github.com/xianmin/hugo-theme-jane.git themes/jane
cp -r themes/jane/exampleSite/content ./
cp themes/jane/exampleSite/config.toml ./

```

 * Test hugo at local.

 ``` bash
	
hugo server

```

* You can access [http://localhost:1313](http://localhost:1313) to test your blog in local.


#### Change configure

 * Modifiy vi config.toml
 
``` toml
#Change URL
baseURL = "https://blog.stevedong.com"

#Change Title
title = "Steve Dong - A System Administrator's Blog"

#Change theme to jane
theme = "jane"

#How many pages will be displayed on homepage.
pageinate = 15

# Display how many pages will be displayed on archive page.
archive-paginate = 50

#RSS limite.
rssLimit = 50

#google analytics related
googleAnalytics= "UA-xxxxxxxx-X"

# Chang author
name = "Steve Dong"

 # Commit the meanu "External Link"
 #[[menu.main]]
 #  name = "MyGitHub"
 #  weight = 50
 #  url = "https://echowings.github.com"
 
 #Change Site creation time
 
 [params]
 	since = "2018"
 	
	logoTitle= "Steve Dong"
	keywords = ["network", "devops', "System Administrator"]
```

### Add syntax highlight

`vi config.toml`

add

```toml

pygmentsCodefences = true #高亮markdown的代码块
pygmentsCodefencesGuessSyntax = true #高亮markdown中没有标注语言的代码块
pygmentsStyle = 'manni' #高亮主题

``` 
	
 
## Deploy your blog to github

 * Add repo of blog website.

  ``` bash 
   
git submodule add --force -b \ 
	master git@github.com:echowings/echowings.github.io.git public  
	
```

 * Run hugo to generate your blog website. 
	
	``` shell
	
	hugo server
	
	```
	
 * publish your website
	
	``` bash
	
    cd public
    git add.
    git commit -m "YOUR COMMIT MESSAGE"
    git push origin master
    
    ```
    
 * Backup your hugo blog to github 

	``` bash
	
    cd ..
    git add.
    git commit -m "YOUR COMMIT MESSAGE"
    git push origin master
    
	```

### Add a comment
Since theme jane has support utterances, we just need to enable utterances apps in github and configure it in config.toml in hugo.

* Create a repo to storage comment. like “blogcomment”.
* enable utternaces apps.

	1. click [utterances app](https://github.com/apps/utterances)
	2. Auth utterances to your github repo “blogcomment”.

* edit `blogs/config.toml` and append follow lines.


  ```toml
	
 [params.utteranc]
      enable = true
      repo="echowings/blogcomment"
      issueTerm="pathname"
      
```

### Download git and rebuild blog environment


``` zsh
git pulll git@github.com:echowings/blogs.git

#if public already exsited  run `rm public`
git submodule add --force -b master git@github.com:echowings/echowings.github.io.git public

```

## Reference

1. [utterances](https://utteranc.es/)
2. [hugo-theme-jane/dev-config.toml](https://github.com/xianmin/hugo-theme-jane/blob/master/exampleSite/config.toml)
3. [如何在github.io搭建Hugo博客站](https://keysaim.github.io/post/blog/deploy-hugo-blog-in-github.io/) 
4. [用chroma进行语法高亮](https://steemkr.com/cn/@heyeshuang/chroma-typography-js)

