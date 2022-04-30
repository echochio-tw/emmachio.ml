# Limitless Ping - Peter's personal blog

## 建置

產生靜態網頁。

```bash
docker run --rm -v $(PWD):/srv/jekyll -v $(pwd)/vendor/bundle:/usr/local/bundle -it jekyll/jekyll jekyll build
```

* `-v $(PWD):/srv/jekyll` : 將 Host 中的檔案丟入 Container 中
* `-v $(pwd)/vendor/bundle:/usr/local/bundle` : 將 Container 中建置使用的檔案保留在 Host 中做 Cache，提升建置速度

## 寫文章

修改文章時可以即時看到修改後的網頁。

```bash
docker run --rm -v $(PWD):/srv/jekyll -v $(pwd)/vendor/bundle:/usr/local/bundle -it -p 4000:4000 jekyll/jekyll jekyll serve
```

* `-p 4000:4000` : Container 中的 4000 port 對應至 Host 的 4000 port

直接瀏覽 localhost:4000 看建置後的頁面。
