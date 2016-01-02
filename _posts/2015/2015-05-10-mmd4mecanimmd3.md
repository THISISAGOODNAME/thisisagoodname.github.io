---
layout: post
title: "MMD4Mecanim系统研究(三)"
description: "将PMX2FBX生成的fbx文件导入maya中"
category: Unity3D 
tags: [Unity3D,MMD4Mecanim,maya]
---

* Table of Contents
{:toc}

<script src="/js/SyntaxHighlighter/jquery.highlighter.js?v=20091222" type="text/javascript"></script>
<script src="/js/SyntaxHighlighter/highlighter.js?v=20091222" type="text/javascript"></script>


##PMX模型导入U3D使用,并使用PMX2FBX,对PMX模型完成转换，再导入到maya中,形成另外的工作流

&#160; &#160; &#160; &#160;玩过unity和MMD的应该都知道MMD4Mecanim这个东东吧，几乎可以完美把PMX/PMD模型转成fbx导入unity。但是这个fbx不带贴图的，unity是通过对于的xml读取贴图，所以导入maya是一片灰色。
&#160; &#160; &#160; &#160;网上能够较好应付PMD/PMX模型的也只有blender和C4D了，但blender操作和主流三维软件还是有些区别，而C4D的MMD tools总是丢材质球，简直蛋疼。
&#160; &#160; &#160; &#160;转换之后，可以获得带贴图的FBX文件，再次导出的话，模型也

<!-- more -->

###1.导入MMD4Mecanim插件，并导入模型

&#160; &#160; &#160; &#160;导入模型的方法我在我的[另一篇文章](./mmd4mecanimmd.html)中已经详细的记录过了在此不再赘描述。

###2.使用Maya的MMD4MecanimImport插件

&#160; &#160; &#160; &#160;MMD4MecanimImport是博客园上一位叫做[高町☆魔理沙](http://home.cnblogs.com/u/marisa/)的大神制作的，[原文链接](http://www.cnblogs.com/marisa/p/4131746.html)。

###3.maya导入MMD4Mecanim生成的FBX

&#160; &#160; &#160; &#160;安装完成MMD4MecanimImport插件之后，用该插件导入MMD4Mecanim转换的FBX模型。我用的是mac版的maya2014，MMD4Mecanim版本不能太新(15年4月版本以前的，估计是新版本修改的xml文件的一些细节，以后有机会研究maya脚本的话我会修改一下脚本文件)

![导入到maya](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101241316013571876338_001.jpg)


&#160; &#160; &#160; &#160;导入完成，用着色模式查看一下(如果机子配置较低可以考虑用局部渲染看一下效果),然后选择导入的模型，再导出成fbx。保存，看到生成的fbx文件只有几M，相比过去的模型，连模型加贴图有几十M，小了很多。

###4.之后就可以用自己熟悉的工作流了

&#160; &#160; &#160; &#160;我将模型导入到keyshot5中渲染一下

![导入到keyshot5](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101241316013571876338_000.jpg)

更多高级玩法可以去看魔理沙大神的博文原文

PS:MMD4MecanimImport.py源代码如下，我做了微小的修改，使之在mac系统上使用稍微方便一些。



<pre><code>
import pymel.util.path as pmp
import maya.cmds as cmds
import maya.mel as mel
import maya.OpenMayaUI as OpenMayaUI
import shiboken
import os
import shutil
from PySide import QtGui, QtCore
from xml.dom.minidom import parse

class FBXExplorerQtWindow(QtGui.QMainWindow):
	def __init__(self,parent=None):
		super(FBXExplorerQtWindow,self).__init__(parent)
		self.setWindowTitle('MMD4Mecanim Import v0.1')
		self.resize( 800, 800 )
		
		widget = QtGui.QWidget()
		self.setCentralWidget(widget)
		
		mainLayout = QtGui.QVBoxLayout()
		widget.setLayout(mainLayout)
		
		self.line = QtGui.QLineEdit(self)
		self.fileBrowserWidget = QtGui.QWidget(self)
		self.dirModel = QtGui.QFileSystemModel()
		self.dirModel.setRootPath("~/Documents/")
		
		nameFilters = ["*.fbx"]
		self.dirModel.setNameFilters(nameFilters)
		self.dirModel.setNameFilterDisables(False)
		
		self.folderView = QtGui.QTreeView(parent=self)
		self.folderView.setModel(self.dirModel)
		self.folderView.clicked[QtCore.QModelIndex].connect(self.clicked) 
		self.folderView.doubleClicked[QtCore.QModelIndex].connect(self.doubleClicked) 
		
		self.folderView.setColumnWidth(0,250)
		
		mainLayout.addWidget(self.line,1,0)
		mainLayout.addWidget(self.folderView)

	def clicked(self,index):
		index = self.folderView.currentIndex()
		dirPath = pmp(self.dirModel.filePath(index))
		self.line.setText(dirPath.normpath())

	def doubleClicked(self,index):
		index = self.folderView.currentIndex()
		path = pmp(self.dirModel.filePath(index))
		self.openFile(path)

	def openFile(self,filePath):
		types = {".fbx":"FBX"}
		if filePath.ext.lower() not in types.keys():
			return
		fileType = types[filePath.ext.lower()]
		if fileType == "FBX":
			nPos = filePath.rfind('/')
			dirPath = filePath[0:nPos+1]
			fileName = filePath[nPos+1:filePath.rfind('.')]
			fbxFileName = filePath[nPos+1:len(filePath)]
			xmlFileName = fileName + '.xml'
			xmlFilePath = dirPath + xmlFileName
			newDirPath = self.copyFileForMaya(fileName, filePath, xmlFilePath, dirPath)
			
			if os.path.exists(newDirPath):
				newFbxFilePath = newDirPath + fbxFileName
				newXmlFilePath = newDirPath + xmlFileName
				self.processRename(newFbxFilePath, newXmlFilePath)
				self.importFBXFile(newFbxFilePath)
				self.importTexture(newXmlFilePath, newDirPath)
				shutil.rmtree(newDirPath,True)
			return

	def copyFileForMaya(self, fbxFileName, fbxFilePath, xmlFilePath, dirPath):
		newDirName = '.temp_' + fbxFileName
		newDirPath = os.path.join(dirPath, newDirName)
		if not os.path.isdir(newDirPath):
			os.makedirs(newDirPath)
		
		newDirPath += '/'	
		if not os.path.exists(newDirPath):
			print newDirPath + ' does not exist!'
			return ''
		
		shutil.copy(fbxFilePath, newDirPath)
		shutil.copy(xmlFilePath, newDirPath)
		return newDirPath

	def modify(self, filePath, sourceStr, targetStr):
		print 'modify ' + filePath + ' from ' + sourceStr + ' to ' + targetStr
		inputFile = open(filePath)
		lines = inputFile.readlines()
		inputFile.close()

		outputFile = open(filePath, 'w')
		for line in lines:
			if not line:
				break
			if sourceStr in line and not targetStr in line:
				nPos = line.find(sourceStr)
				temp1 = line[0:nPos]
				temp2 = line[nPos+len(sourceStr):len(line)]
				temp = temp1 + targetStr + temp2
				outputFile.write(temp)
			else:
				outputFile.write(line)

		outputFile.close()

	def formatName(self, name):
		newName = ''
		if '.' in name:
			nPos = name.find('.')
			newName = 'mat_' + name[0:nPos]
		else:
			newName = 'mat_' + name
		return newName
		
	def modifyMaterialName(self, fbxFilePath):
		inputFbxFile = open(fbxFilePath)
		inputFbxLines = inputFbxFile.readlines()
		inputFbxFile.close()
		outputFbxFile = open(fbxFilePath, 'w')
		tag1 = 'Material::'
		tag2 = ';Material::'
		for line in inputFbxLines:
			if not line:
				break
			if tag2 in line:
				nPos1 = line.find(tag2)
				temp1 = line[0:nPos1]
				temp2 = line[nPos1+len(tag2):len(line)]
				nPos2 = temp2.find(', Model::')
				temp3 = temp2[nPos2:len(temp2)]
				materialName = self.formatName(temp2[0:nPos2])
				temp = temp1 + tag2 + materialName + temp3
				outputFbxFile.write(temp)
			elif tag1 in line:
				nPos1 = line.find(tag1)
				temp1 = line[0:nPos1]
				temp2 = line[nPos1+len(tag1):len(line)]
				nPos2 = temp2.find('"')
				temp3 = temp2[nPos2:len(temp2)]
				materialName = self.formatName(temp2[0:nPos2])
				temp = temp1 + tag1 + materialName + temp3
				outputFbxFile.write(temp)
			else:
				outputFbxFile.write(line)
		outputFbxFile.close()
		
	def processRename(self, fbxFilePath, xmlFilePath):
		self.modifyMaterialName(fbxFilePath)
		self.modify(xmlFilePath, '<materialName>', '<materialName>mat_')
		self.modify(xmlFilePath, '<fileName>', '<fileName>../')
		print 'Material rename completed!'

	def importFBXFile(self, filePath):
		nPos = filePath.rfind('/')
		fileName = filePath[nPos+1:filePath.rfind('.')]
		importedFile = cmds.file(filePath, i=True, type='FBX', ignoreVersion=True, ra=True, rdn=True, mergeNamespacesOnClash=False, namespace=fileName)
		print importedFile + ' import completed!'
	
	def importTexture(self, filePath, dirPath):
		dom = parse(filePath)
		
		texFileNames = dom.getElementsByTagName("fileName")
		textures = []
		for i, texFileName in enumerate(texFileNames) :
			textures.append(dirPath.encode('shift-jis') + texFileName.childNodes[0].data.encode('shift-jis'))
			if cmds.objExists('file' + str(i+1)) :
				continue
			else:
				fileNode = cmds.shadingNode('file', asTexture=True)
				cmds.setAttr(fileNode + '.fileTextureName', textures[i], type="string")
		print textures
		
		materialNames = dom.getElementsByTagName("materialName")	
		materialID = []
		for i, materialName in enumerate(materialNames) :
			materialID.append(materialName.childNodes[0].data)
		print materialID
		
		textureIDs = dom.getElementsByTagName("textureID")
		materialTexID = []
		for i, textureID in enumerate(textureIDs) :
			materialTexID.append(textureID.childNodes[0].data)
		print materialTexID
		
		for i, iMatID in enumerate(materialID) :
			iTexID = int(materialTexID[int(i)])+1
			if(iTexID <= 0):
				continue
			try:
				cmds.connectAttr('file%s.outColor' %iTexID, '%s.color' %iMatID)
				print iMatID
			except:
				continue

def show_explorer(*args):
	ptr = OpenMayaUI.MQtUtil.mainWindow()
	widget = shiboken.wrapInstance(long(ptr),QtGui.QWidget)
	explorerWin = FBXExplorerQtWindow(widget)
	explorerWin.show()

def show_help(*args):
	cmds.confirmDialog(title = "Help", message = "\
Steps to import:\n\
1. Convert PMD/PMX file to FBX file in unity by using MMD4Mecanim.\n\
2. Click 'MMD4Mecanim Import' to browse the FBX file of step1, double click to import.\n\
3. In MotionBuilder, click 'File/open' to import the FBX file of step1 with default configuration (recommend to uncheck the material).\n\
4. In MotionBuilder, select the skeleton (or all the model) you imported at step3, click 'File/Send to Maya/Update Current Scene'.\n\
\n\
Attention:\n\
1. The file name of fbx file and texture files should not be japanese or chinese.\n\
2. You can only import one model at a time, please save your model as the standard fbx file, then create a new scene to import another one.\n\
\n\
Enjoy! >_< \n\
\n\
Author: Takamachi Marisa\n\
Contact: http://weibo.com/u/2832212042",\
	icon = "information")

def customMayaMenu():
	gMainWindow = mel.eval('$temp1=$gMainWindow')
	menus = cmds.window(gMainWindow, q = True, menuArray = True)
	found = False
	
	for menu in menus:
		label = cmds.menu(menu, q = True, label = True)
		if label == "MMD4Mecanim Import":
			found = True
	
	if found == False:
		customMenu = cmds.menu(parent=gMainWindow, label = 'MMD4Mecanim Import')
		cmds.menuItem(parent = customMenu, label = "Import MMD4Mecanim FBX", c = show_explorer)
		cmds.menuItem(parent = customMenu, label = "Help", c = show_help)

# Initialize the plug-in
def initializePlugin(plugin):
	customMayaMenu()

# Uninitialize the plug-in
def uninitializePlugin(plugin):
	pass
</code></pre>