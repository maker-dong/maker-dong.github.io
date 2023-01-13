# linux清除缓存

linux清除缓存：需要root权限
````
sync
echo 3 >/proc/sys/vm/drop_caches
````

上面的echo 3 是清理所有缓存
- echo 0 是不释放缓存
- echo 1 是释放页缓存
- ehco 2 是释放dentries和inodes缓存
-  echo 3 是释放 1 和 2 中说道的的所有缓存