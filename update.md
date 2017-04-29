# 更新译文

`en` 分支是原文，`master` 分支是译文。先更新原文，再合并到译文，然后更新译文。

原文 [commit history](https://github.com/npm/node-semver/commits/master/README.md)

```bash
git checkout en
curl -o README.md "https://raw.githubusercontent.com/npm/node-semver/master/README.md"
git commit -am 'update original'

git checkout master
git merge en
git mergetool
git commit -m 'update translation'
git push
```
