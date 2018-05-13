> 我在简书写博客, 会使用markdown格式, 然后把相关资源传到百度网盘, 但最近发现百度网盘的资源经常会被取消分享, 为了彻底解决这个问题, 我把选择把资源传到Github, 博客本身的markdown格式可以作为README.md

#### 于是我写了一个脚本,让程序自动完成这项工作:
> ![目录结构](https://upload-images.jianshu.io/upload_images/3203841-67bc879056df883b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 运行结果

> ![运行结果](https://upload-images.jianshu.io/upload_images/3203841-9c1b5ef5e80b9001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> ![运行后](https://upload-images.jianshu.io/upload_images/3203841-0bcef2ecac3c4419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 源码:

###### 1. 主脚本文件

```python
# 本脚本需要提供的资源信息:
## 信息1: github账户及密码, 
## 信息2: 新建的github仓库名称(驼峰式英文名)
## 信息3: 简书的.md文档
## 信息4: 需要上传的其他文件资源(单个文件资源不超过100M)

# 本脚本完成三个任务
## 任务1: 根据用户提供的仓库名创建github仓库, 
## 任务2: 将简书.md文档作为README.md上传到github
## 任务3: 将其他文件资源(单文件不超过100M)上传到github(原来我一直放到百度盘, 后来发现百度盘分享经常挂掉, 就放弃了百度)

# 环境要求
## 1. 已经安装curl
## 2. 已经安装git

import os
import json

def getInfo():
	info = {}
	with open("./inputInfo.txt", 'r') as f:
		jsonStr = ''
		lines = f.readlines()
		# 过滤注释, 生成json格式
		for line in lines:
			if '#' not in line:
				jsonStr += line
		info = json.loads(jsonStr)

	return info

# 在github创建远程仓库
def CreateRepository(info):
	GitHubUserName = info['GitHubUserName']
	GitHubPassWord = info['GitHubPassWord']
	GitHubRepositoryName = info['GitHubRepositoryName']

	new_command = 'curl -i -u ' + '\'' +GitHubUserName + ':' + GitHubPassWord + '\'' +' -d ' + '\''+ '{"name": ' + '\"'+GitHubRepositoryName +'\"'+ ', ' + '"auto_init": ' + 'true, ' + '"private": ' + 'false, ' + '"gitignore_template": ' + '"nanoc"}' + '\'' + ' https://api.github.com/user/repos'
	result = os.popen(new_command).readlines()
	if ('HTTP/1.1 201 Created\n' in result):
		print("创建成功")
		return True
	else:
		return False
	
def GetRepository(info):

	GetAllRepCommand = 'curl -i -u ' + '\'' + info['GitHubUserName'] + ':' + info['GitHubPassWord'] +'\'' + ' https://api.github.com/user/repos'
	print(GetAllRepCommand)
	result = os.popen(GetAllRepCommand).readlines()
	keyWord = info['GitHubUserName']+'/'+info['GitHubRepositoryName']
	# 判断仓库是否创建成功
	if not (keyWord in str(result)):
		return
	# 获取仓库到同级目录下
	# git@github.com:zhaoolee/ChatRoom.git
	GetRepCommand = 'git clone git@github.com:' +  keyWord + '.git'

	# 将仓库获取到本地
	result = os.popen(GetRepCommand).readlines()

# 将资源文件放入仓库
def FillRepository(info):
	AllFileName = os.listdir('./')
	PreReadMeFile = ''
	for FileName in AllFileName:
		if FileName[-3:] == '.md':
			PreReadMeFile = FileName

	# 将md文件替换原有的README.md
	ReplaceMdFileCommand = 'cp ./' + PreReadMeFile + ' ./'+ info['GitHubRepositoryName'] + '/README.md'
	print("==>", ReplaceMdFileCommand, "<==")
	result = os.popen(ReplaceMdFileCommand).readlines()

	# 将resource文件夹, 放入仓库中
	RemoveResourceCommand = 'cp -r resource ' + './' + info['GitHubRepositoryName']
	print('RemoveResourceCommand==>', RemoveResourceCommand)
	result = os.popen(RemoveResourceCommand).readlines()

# 将文件提交到仓库
def PushRepository(info):
	inputRepository = 'cd ' + info['GitHubRepositoryName']
	addCommand = 'git add .'
	result = os.popen(inputRepository+'\n'+addCommand).readlines()
	commitCommand = 'git commit -m "完成项目的初始化"'
	result = os.popen(inputRepository+'\n'+commitCommand).readlines()
	pushCommand = 'git push'
	result = os.popen(inputRepository+'\n'+pushCommand).readlines()
	print("完成")


def main():
	# 获取信息
	info = getInfo()
	# 创建仓库, 并通过ssh保存到本地
	CreateRepository(info)
	# 将仓库git到本地
	GetRepository(info)
	# 将资源文件转入代码仓库
	FillRepository(info)
	# 将资源提交到仓库
	PushRepository(info)

if __name__ == '__main__':
	main()
```
###### 2. 配置文件
```
{
	# 用户名
	"GitHubUserName": "zhaoolee", 
	# 用户密码
	"GitHubPassWord": "github", 
	# 将要新建的仓库
	"GitHubRepositoryName": "TestCreateRep"
}
```
> 为便于管理, 相关资源整合到一张独立的帖子，链接如下:
[http://www.jianshu.com/p/4f28e1ae08b1](https://www.jianshu.com/p/4f28e1ae08b1)
