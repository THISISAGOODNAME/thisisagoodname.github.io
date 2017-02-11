---
layout: post
title: "qt中使用opengl并设置openGL版本"
description: "qt中使用opengl并设置openGL版本"
category: Qt
tags: [c++,Qt,opengl]
---

&#160; &#160; &#160; &#160;其实写glitter的那篇博文的时候，就想把Qt初始化OpenGL的方法也记录一下的，结果当时忘记了，现在才想起来。现在补写一下。

<!-- more -->

* Table of Contents
{:toc}

# Qt中的openGL

&#160; &#160; &#160; &#160;Qt在Qt4的时代，就提供了Qt OpenGL模块，来提供对OpenGL的支持。到了Qt5时代，Qt OpenGL模块已经不建议使用，OpenGL/OpenGLES相关的类被移动到了Qt GUI模块中。在Qt Widgets模块中的QOpenGLWidget类提供了一个可以渲染OpenGL图形的部件，通过此组件可以轻松的将OpenGL surface添加到Qt应用程序之中。

&#160; &#160; &#160; &#160;QOpenGLWidget类使用起来和QWidget相同，集成该类，然后和其他QWidget一样即可。QOpenGLWidget提供了三个虚函数，可以在子类中重新实现他们来执行典型的OpenGL任务：

1. initializeGL():设置OpenGL资源和状态。该函数只在第一次调用resizeGL()或paintGL()前被调用一次。
2. resizeGL():设置OpenGL的视口、投影等。每次部件大小改变时都会调用该函数。
3. paintGL():渲染OpenGL场景。每当部件需要更新时都会调用该函数。

# 实例

## 1. 新建工程

&#160; &#160; &#160; &#160;我使用Qt5，新建一个Qt项目，然后打开项目文件(.pro)，添加如下行：

```Makefile
QT += widgets
```

## 2. 创建类

&#160; &#160; &#160; &#160;新建一个名为MyOpenGLWidget的类，头文件修改如下：

```c++
#ifndef MYOPENGLWIDGET_H
#define MYOPENGLWIDGET_H

#include <QOpenGLWidget>
#include <QOpenGLFunctions>

class QOpenGLShaderProgram;
class MyOpenGLWidget : public QOpenGLWidget, protected QOpenGLFunctions
{
    Q_OBJECT

public:
    explicit MyOpenGLWidget(QWidget *parent = 0);
    ~MyOpenGLWidget();

protected:
    void initializeGL();
    void paintGL();
    void resizeGL(int width, int height);

private:
    QOpenGLShaderProgram * program;
    void printContextInformation();
};

#endif // MYOPENGLWIDGET_H
```

&#160; &#160; &#160; &#160;继承QOpenGLWidget类的同时还继承QOpenGLFunctions的目的，是可以直接使用QOpenGLFunctions中的OpenGL函数，而不需要创建QOpenGLFunctions手工查询openGL函数。

&#160; &#160; &#160; &#160;接下来在c++中实现各函数：

```c++
#include "myopenglwidget.h"
#include <QOpenGLShaderProgram>
#include <QDebug>
#include <iostream>

MyOpenGLWidget::MyOpenGLWidget(QWidget *parent)
    : QOpenGLWidget(parent)
{
}

MyOpenGLWidget::~MyOpenGLWidget()
{

}

void MyOpenGLWidget::initializeGL()
{
    // 为当前环境初始化OpenGL函数
    initializeOpenGLFunctions();
    printContextInformation();

    // 创建顶点着色器
    QOpenGLShader *vshader = new QOpenGLShader(QOpenGLShader::Vertex, this);
    const char *vsrc =
            "void main() {                             \n"
            "   gl_Position = vec4(0.0, 0.0, 0.0, 1.0);\n"
            "}                                         \n";
    vshader->compileSourceCode(vsrc);
    // 创建片段着色器
    QOpenGLShader *fshader = new QOpenGLShader(QOpenGLShader::Fragment, this);
    const char *fsrc =
            "void main() {                              \n"
            "   gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);\n"
            "}                                          \n";
    fshader->compileSourceCode(fsrc);

    // 创建着色器程序
    program = new QOpenGLShaderProgram;
    program->addShader(vshader);
    program->addShader(fshader);
    program->link();
    program->bind();
}

void MyOpenGLWidget::resizeGL(int , int )
{
}

void MyOpenGLWidget::paintGL()
{

    // 绘制
    glDrawArrays(GL_POINTS, 0, 1);
}

void MyOpenGLWidget::printContextInformation()
{
    QString glType;
    QString glVersion;
    QString glProfile;

    // Get Version Information
    glType = (context()->isOpenGLES()) ? "OpenGL ES" : "OpenGL";
    glVersion = reinterpret_cast<const char*>(glGetString(GL_VERSION));

    // Get Profile Information
#define CASE(c) case QSurfaceFormat::c: glProfile = #c; break
    switch (format().profile())
    {
    CASE(NoProfile);
    CASE(CoreProfile);
    CASE(CompatibilityProfile);
    }
#undef CASE

    // qPrintable() will print our QString w/o quotes around it.
    qDebug() << qPrintable(glType) << qPrintable(glVersion) << "(" << qPrintable(glProfile) << ")";
}
```

&#160; &#160; &#160; &#160;其中，printContextInformation函数是用来显示openGL版本的辅助函数。我使用的着色器对应低版本的openGL，需要开启兼容模式才能正确显示。

## 3. 在程序中添加MyOpenGLWidget

&#160; &#160; &#160; &#160;修改main.cpp如下：

```c++
#include "myopenglwidget.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    // 设置 OpenGL 版本信息
    // 注意: format 必须在 show() 调用前设置
    QSurfaceFormat format;
    format.setRenderableType(QSurfaceFormat::OpenGL);
    format.setProfile(QSurfaceFormat::CoreProfile);
//    format.setProfile(QSurfaceFormat::CompatibilityProfile);
    format.setVersion(4,3);

    MyOpenGLWidget w;
    w.setFormat(format);
    w.resize(400, 300);
    w.show();

    return a.exec();
}
```

&#160; &#160; &#160; &#160;QSurfaceFormat可以设置openGL版本。

# QSurfaceFormat的一些说明

&#160; &#160; &#160; &#160;在使用`QSurfaceFormat format;`声明format变量之后。需要在OpenGLWidget实例使用show()方法钱用setFormat()QSurfaceFormat可以设置openGL版本。

&#160; &#160; &#160; &#160;`format.setRenderableType()`可以设置渲染使用的程序，不光支持openGL，还支持OpenGLES和OpenVG。

&#160; &#160; &#160; &#160;`format.setProfile()`可以设置openGL使用只支持可编程管线的核心模式还是支持固定管线的兼容模式。

&#160; &#160; &#160; &#160;`format.setVersion()`设置使用openGL的版本，第一位是主版本号，第二位是次版本号。

&#160; &#160; &#160; &#160;注意：不知道是Qt的问题还是NVIDIA驱动的问题，如果`setProfile()`的参数是`QSurfaceFormat::CompatibilityProfile`，那么使用的openGL版本必然为`OpenGL 4.5.0 NVIDIA <驱动版本号> ( CompatibilityProfile )`。不过NV的openGL驱动相对宽松，有非常好的向下兼容性。同时，只有使用OpenGL 3.1版本以上时，才能选用openGL核心模式，3.0以下openGL版本会自动变为`OpenGL 4.5.0 NVIDIA <驱动版本号> ( CompatibilityProfile )`，当然并不影响程序运行就是了。
