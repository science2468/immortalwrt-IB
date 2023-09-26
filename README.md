GitHub Actions immortalwrt-Image-Builder 最简单的\
https://github.com/1715173329/blog/issues/8 天灵出品IB使用教程\
https://blog.imouto.in/#/posts/8 天灵出品IB使用教程\
packages=" "  双引号之间是添加包的，比如 make packages="kmod-alx" 这就是增加包的;make packages="-kmod-alx"这就是去除包的。包和包之间是需要一个空格的。\
示例命令：make image PROFILE="profile-name" PACKAGES="pkg1 pkg2 pkg3 -pkg4 -pkg5 -pkg6" FILES="files"
