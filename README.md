This is the hexo code repo for my hexo blog.

blog address: https://qiusiyuan.github.io/

blog theme: [3-hexo](https://github.com/yelog/hexo-theme-3-hexo)

### special notice

This repo is using travis github page deploy.

A single commit will trigger travis to deploy blog on to https://qiusiyuan.github.io/

Travis dashboard:

https://travis-ci.com/

login with github

(deprecated) To deploy blog,
```
hexo clean
hexo g
hexo d
```

when push: `git push origin` to avoid push to blog repo

if deploy failed, remove .deploy_git
`rm -rf .deploy_git`

to use 3-hexo:
* For code, must leave a blank line at the end of each code block.