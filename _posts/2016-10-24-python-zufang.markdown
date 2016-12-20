---
layout:     post
title:      "Python 爬虫找房大法"
subtitle:   "使用 python 爬虫对豆瓣进行爬取"
date:       2016-10-24
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - python
    - 爬虫
---

> 记一次租房经历


## Life is short. You need Python.

打算寒假去实习，租房是件头疼的事，房源很多很多，骗子更是不少，经常逛豆瓣无中介租房小组，感觉质量挺高的，但是租房信息很多，根本看不过来，每天都要花大部分时间找房。突然想到用python写个爬虫，这就能省很多事了，T T


#### 抓取 html 源代码

	def getsource(self,url):
	        html = requests.get(url)
	        return html.text

#### 获取 url 修改页面参数，存入page_group

	def changepage(self, url, total_page):
        now_page = int(re.search('start=(\d+)', url, re.S).group(1))
        page_group = []
        count = 0
        for i in range(now_page, total_page):
            count = count + 50
            link = re.sub('start=\d+', 'start=%s'%count, url, re.S)
            page_group.append(link)
        return page_group


#### 使用正则获取指定标签信息

	def find_text(self, source):
	        everyhose = re.findall('<td class="title">(.*?)</td>', source, re.S)
	        return everyhose

#### 将 eachhouse 数组里每一条记录用正则表达式分离

	def getinfo(self, eachhouse):
	        info = {}
	        info['url'] = re.search(r'<a href="(.*?)"', eachhouse, re.S).group(1)
	        info['title'] = re.search(r'title="(.*?)" class', eachhouse, re.S).group(1)
	        return info


#### 存入文件

	def saveinfo(self, classinfo):
	        f = open('3.txt', 'a')
	        for each in classinfo:
	            f.writelines(each['url'] + '\n')
	            f.writelines(each['title'] + '\n' + '\n')
	        f.close()


### 源代码

	#-*_coding:utf8-*-
	import requests
	import re
	import sys
	reload(sys)
	sys.setdefaultencoding("utf-8")
	
	class DouBan(object):
	    def __init__(self):
	        print u'开始爬取内容。。。'
	
	    def getsource(self,url):
	        html = requests.get(url)
	        return html.text
	
	    def changepage(self, url, total_page):
	        now_page = int(re.search('start=(\d+)', url, re.S).group(1))
	        page_group = []
	        count = 0
	        for i in range(now_page, total_page):
	            count = count + 50
	            link = re.sub('start=\d+', 'start=%s'%count, url, re.S)
	            page_group.append(link)
	        return page_group
	
	    def getinfo(self, eachhouse):
	        info = {}
	        info['url'] = re.search(r'<a href="(.*?)"', eachhouse, re.S).group(1)
	        info['title'] = re.search(r'title="(.*?)" class', eachhouse, re.S).group(1)
	        return info
	
	    def saveinfo(self, classinfo):
	        f = open('3.txt', 'a')
	        for each in classinfo:
	            f.writelines(each['url'] + '\n')
	            f.writelines(each['title'] + '\n' + '\n')
	        f.close()
	
	    def find_text(self, source):
	        everyhouse = re.findall('<td class="title">(.*?)</td>', source, re.S)
	        return everyhouse
	
	if __name__ == '__main__':
	
	    pages = []
	    file = []
	    url = 'https://www.douban.com/group/zhufang/discussion?start=0'
	    dou = DouBan()
	    pages = dou.changepage(url, 300)
	    for page in pages:
	        print ("正在爬取" + page)
	        page_info = dou.find_text(dou.getsource(page))
	        for info in page_info:
	            infos = {}
	            infos = dou.getinfo(info)
	            file.append(infos)
	        dou.saveinfo(file)