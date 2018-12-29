---
layout: post
title: "openGL program object 研究"
description: "openGL program object 研究"
image: ""
date: 2018-12-29 15:35:48
categories: 研究
tags: [研究]
---

&nbsp; &nbsp; &nbsp; &nbsp;openGL program object是一个比较特殊的对象，因为他的老对手DX没有(DX的shader直接set在DeviceContext上)，他的后辈vulkan和metal没有(pipeline object上)。今天写一篇文章，学习一下openGL program object的一些东西。

<!-- more -->
* Table of Contents
{:toc}

# 创建openGL program

&nbsp; &nbsp; &nbsp; &nbsp;openGL program的创建其实很灵活，可以从glsl源码创建，也可以从编译好的openGL binary program创建，还可以使用SPIR-V文件创建(openGL 4.5 +)

## 1. Use this to load and compile the shader program.

&nbsp; &nbsp; &nbsp; &nbsp;使用glsl创建program，就是compile shader文件，attach shader，link program。

```c++
GLuint programHandle;

void compileShaderProgram() {
    std::cout << "Compiling shader program" << std::endl;

	//////////////////////////////////////////////////////
	/////////// Vertex shader //////////////////////////
	//////////////////////////////////////////////////////

	// Load contents of file
	std::ifstream inFile("shader/basic.vert.glsl");
	if (!inFile) {
		fprintf(stderr, "Error opening file: shader/basic.vert.glsl\n");
		exit(1);
	}

	std::stringstream code;
	code << inFile.rdbuf();
	inFile.close();
	string codeStr(code.str());

	// Create the shader object
	GLuint vertShader = glCreateShader(GL_VERTEX_SHADER);
	if (0 == vertShader) {
		std::cerr << "Error creating vertex shader." << std::endl;
		exit(EXIT_FAILURE);
	}

	// Load the source code into the shader object
	const GLchar* codeArray[] = { codeStr.c_str() };
	glShaderSource(vertShader, 1, codeArray, NULL);

	// Compile the shader
	glCompileShader(vertShader);

	// Check compilation status
	GLint result;
	glGetShaderiv(vertShader, GL_COMPILE_STATUS, &result);
	if (GL_FALSE == result) {
		std::cerr << "Vertex shader compilation failed!" << std::endl;
		std::cerr << getShaderInfoLog(vertShader) << std::endl;
        exit(EXIT_FAILURE);
	}

	//////////////////////////////////////////////////////
	/////////// Fragment shader //////////////////////////
	//////////////////////////////////////////////////////

	// Load contents of file into shaderCode here
	std::ifstream fragFile("shader/basic.frag.glsl");
	if (!fragFile) {
		fprintf(stderr, "Error opening file: shader/basic.frag.glsl\n");
		exit(1);
	}

	std::stringstream fragCode;
	fragCode << fragFile.rdbuf();
	fragFile.close();
	codeStr = fragCode.str();

	// Create the shader object
	GLuint fragShader = glCreateShader(GL_FRAGMENT_SHADER);
	if (0 == fragShader) {
		fprintf(stderr, "Error creating fragment shader.\n");
		exit(1);
	}

	// Load the source code into the shader object
	codeArray[0] = codeStr.c_str();
	glShaderSource(fragShader, 1, codeArray, NULL);

	// Compile the shader
	glCompileShader(fragShader);

	// Check compilation status
	glGetShaderiv(fragShader, GL_COMPILE_STATUS, &result);
	if (GL_FALSE == result) {
        std::cerr << "Fragment shader compilation failed!" << std::endl;
        std::cerr << getShaderInfoLog(vertShader) << std::endl;
        exit(EXIT_FAILURE);
	}

	linkMe(vertShader, fragShader);
}

void linkMe(GLint vertShader, GLint fragShader)
{
    // Create the program object
    programHandle = glCreateProgram();
    if(0 == programHandle) {
        std::cerr << "Error creating program object." << std::endl;
        exit(EXIT_FAILURE);
    }

    // Bind index 0 to the shader input variable "VertexPosition"
    //glBindAttribLocation(programHandle, 0, "VertexPosition");
    // Bind index 1 to the shader input variable "VertexColor"
    //glBindAttribLocation(programHandle, 1, "VertexColor");

    // Attach the shaders to the program object
    glAttachShader( programHandle, vertShader );
    glAttachShader( programHandle, fragShader );

    // Link the program
    glLinkProgram( programHandle );

    // Check for successful linking
    GLint status;
    glGetProgramiv( programHandle, GL_LINK_STATUS, &status );
    if (GL_FALSE == status) {
        std::cerr << "Failed to link shader program!" << std::endl;
        std::cerr << getProgramInfoLog(programHandle) << std::endl;
        exit(EXIT_FAILURE);
    }

	// Clean up shader objects
	glDetachShader(programHandle, vertShader);
	glDetachShader(programHandle, fragShader);
	glDeleteShader(vertShader);
	glDeleteShader(fragShader);

    glUseProgram( programHandle );
}
```

&nbsp; &nbsp; &nbsp; &nbsp;在 `glUseProgram` 之前，我调用了 `glDetachShader` 。其实这样完全没有关系，对于openGL program，只有在调用 `glLinkProgram` 才会真正使用attach的shader生成新的program。换句话说，只要不调用 `glLinkProgram`，program上的shader可以自由的attach和detach，不会对功能有任何影响。

&nbsp; &nbsp; &nbsp; &nbsp;对于某些硬件，支持单shader program(比如只有vertex shader没有fragment shader)。这其实非常有用，对于现代渲染管线(F+,CFR)，early-Z pass至关重要，只需要顶点光栅化写depth buffer，完全不需要fragment shader以及之后所有操作。单shader program的作用就凸现出来了(当然和vulkan那种定制pipeline，还有使用compute shader定制pipeline相比还是很原始)。

&nbsp; &nbsp; &nbsp; &nbsp;在上面的程序中，我detach shader之后才执行 delete shader，但是其实shader attach之后就可以删除了。至于detach命令，一般情况下都不会用到。在delete program的时候，会自动detach并delete所有相关shader。而据 openGL 文档描述，在 render context 确认没有对 program 的使用后，也会自动 delete program。

## 2. Load a binary (pre-compiled) shader program.  (file: "shader/program.bin")

&nbsp; &nbsp; &nbsp; &nbsp;使用二进制文件创建 program 的过程非常简单。

```c++
GLuint programHandle;

void loadShaderBinary(GLint format) {
    std::cout << "Loading shader binary: shader/program.bin (format = " << format << ")" << std::endl;

    // Create the program object
	programHandle = glCreateProgram();
	if (0 == programHandle) {
		std::cerr << "Error creating program object." << std::endl;
		exit(EXIT_FAILURE);
	}

	std::ifstream inStream("shader/program.bin", std::ios::binary);
	std::istreambuf_iterator<char> startIt(inStream), endIt;
	std::vector<char> buffer(startIt, endIt);
	inStream.close();
	glProgramBinary(programHandle, format, buffer.data(), buffer.size());

	// Check for successful linking
	GLint status;
	glGetProgramiv(programHandle, GL_LINK_STATUS, &status);
	if (GL_FALSE == status) {
		std::cerr << "Failed to load binary shader program!" << std::endl;
        std::cerr << getProgramInfoLog(programHandle) << std::endl;
        exit(EXIT_FAILURE);
	}
	glUseProgram(programHandle);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;openGL program看似和DXBC差不多，其实差异甚大。对于openGL program，不同制造商甚至同一制造商的不同代产品，都有可能有差异，因此binary最好是在本机第一次运行时保存下来，在第二次运行以后再使用（这个技术在RCPS3，cemu等模拟器中都有使用）。

## 3. Load a SPIR-V shader program. (files: "shader/vert.spv" and "shader/frag.spv")

&nbsp; &nbsp; &nbsp; &nbsp;在openGL 4.5版本中带来了使用SPIR-V shader的功能，个人猜测是给向vulkan迁移的程序一个过渡。

```c++
void loadSpirvShader() {

    std::cout << "Loading SPIR-V shaders." << std::endl;

    GLint status;
    GLuint vertShader = glCreateShader(GL_VERTEX_SHADER);

    {
        std::cout <<  "Loading SPIR-V binary: shader/vert.spv" << std::endl;
        std::ifstream inStream("shader/vert.spv", std::ios::binary);
        std::istreambuf_iterator<char> startIt(inStream), endIt;
        std::vector<char> buffer(startIt, endIt);
        inStream.close();

        glShaderBinary(1, &vertShader, GL_SHADER_BINARY_FORMAT_SPIR_V_ARB, buffer.data(), buffer.size());
    }

    glSpecializeShaderARB( vertShader, "main", 0, 0, 0);

    glGetShaderiv(vertShader, GL_COMPILE_STATUS, &status);
    if( GL_FALSE == status ) {
        std::cerr << "Failed to load vertex shader (SPIR-V)" << std::endl;
        std::cerr << getShaderInfoLog(vertShader) << std::endl;
        exit(-1);
    }

    GLuint fragShader = glCreateShader(GL_FRAGMENT_SHADER);

    {
        std::cout <<  "Loading SPIR-V binary: shader/frag.spv" << std::endl;
        std::ifstream inStream("shader/frag.spv", std::ios::binary);
        std::istreambuf_iterator<char> startIt(inStream), endIt;
        std::vector<char> buffer(startIt, endIt);
        inStream.close();

        glShaderBinary(1, &fragShader, GL_SHADER_BINARY_FORMAT_SPIR_V_ARB, buffer.data(), buffer.size());
    }

    glSpecializeShaderARB( fragShader, "main", 0, 0, 0);

    glGetShaderiv(fragShader, GL_COMPILE_STATUS, &status);
    if( GL_FALSE == status ) {
        std::cerr << "Failed to load fragment shader (SPIR-V)" << std::endl;
        std::cerr << getShaderInfoLog(fragShader) << std::endl;
        exit(-1);
    }

    // Create the program object
    programHandle = glCreateProgram();
    if (0 == programHandle) {
        std::cerr << "Error creating program object." << std::endl;
        exit(1);
    }

    glAttachShader(programHandle, vertShader);
    glAttachShader(programHandle, fragShader);
    glLinkProgram(programHandle);

    glGetProgramiv(programHandle, GL_LINK_STATUS, &status);
    if (GL_FALSE == status) {
        std::cerr << "Failed to link SPIR-V program" << std::endl;
        std::cerr << getProgramInfoLog(programHandle) << std::endl;
        exit(EXIT_FAILURE);
    }

    glUseProgram(programHandle);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;整体过程和使用glsl源码差不多，但其实有个特殊的地方，就是 `glSpecializeShaderARB` ,这个函数可以指定程序的入口函数，也就是说SPIR-V可以类似DX那样，vertex和fragment使用同一个二进制的shader object，只不过入口函数不同。

&nbsp; &nbsp; &nbsp; &nbsp;严格来说，DXBC和SPIR-V功能上几乎完全等价，现在也有某些开源项目，来实现DXBC/DXIL和SPIR-V的双向转换(比如龚敏敏大神的[Dilithium](https://github.com/gongminmin/Dilithium))。

&nbsp; &nbsp; &nbsp; &nbsp;其实就算没有openGL 4.5，也可以使用 SPIRV-CROSS 将SPIR-V转成GLSL/ESSl shader给低版本的openGL和openGL ES使用。

## 4. Save a binary (pre-compiled) shader program. (file: "shader/program.bin")

```c++
void writeShaderBinary() {
    GLint formats;
    glGetIntegerv(GL_NUM_PROGRAM_BINARY_FORMATS, &formats);
    std::cout << "Number of binary formats supported by this driver = " << formats << std::endl;

    if( formats > 0 ) {
        GLint length;
        glGetProgramiv(programHandle, GL_PROGRAM_BINARY_LENGTH, &length);
        std::cout << "Program binary length = " << length << std::endl;

        std::vector<GLubyte> buffer(length);
        GLenum format;
        glGetProgramBinary(programHandle, length, NULL, &format, buffer.data());
        std::string fName("shader/program.bin");

        std::cout << "Writing to " << fName << ", binary format = " << format << std::endl;
        std::ofstream out(fName.c_str(), std::ios::binary);
        out.write( reinterpret_cast<char *>(buffer.data()), length );
        out.close();
    } else {
        std::cout << "No binary formats supported by this driver.  Unable to write shader binary." << std::endl;
    }
}
```

&nbsp; &nbsp; &nbsp; &nbsp;之前提到的3种方法创建的program均可以保存binary program。

# openGL program 到 vulkan pipeline

&nbsp; &nbsp; &nbsp; &nbsp;vulkan pipeline作为后发，肯定是比openGL program要先进的。就个人认为，vulkan的pipeline，相当于openGL的program和DX11的pipeline的加和。

&nbsp; &nbsp; &nbsp; &nbsp;vulkan pipeline既包含的openGL program管理各个编程阶段shader，检查变量可用性，剔除无用变量，优化shader的代码的责任，又包含DX11 pipeline设置各个render state属性，设置attribute和uniform变量，管理各个render state某些功能开启和关闭情况的功能。并且vulkan pipeline还能复用，并像openGL program一样存成缓存文件下次程序运行时使用。

&nbsp; &nbsp; &nbsp; &nbsp;可以说vulkan/metal中的pipeline，是对openGL program和DX pipeline的继承和发展。



