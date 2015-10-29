---
layout: post
title: 通过CMake支持Qt5的外部资源文件
category : CMake
tags : [CMake，Qt，Qt5]
---
{% include JB/setup %}

最近因为某些原因需要使用到Qt5的外部资源文件，之前一直使用的都是编译到程序内部的资源文件。

在CMake中使用内部的资源文件非常方便，因为Qt5提供了CMake的实现：

    qt5_add_resources(outfiles inputfile ... OPTIONS ...)
	
最后只要将`outfiles`传递给`target`就可以了。

同事找到github上也有个兄弟有这个需求，他自己做了一个CMake函数用于生成`rcc`文件，并且给出了为什么要自己实现的理由：
> Even if forcing this function to pass the -binary option to rcc tool, which is possible, the result is that the linker tries to link also these binary resource files, which obviously is not a good idea.

刚开始没多想，就直接把他的实现直接拿过来用了，在写这篇文档的时候仔细想了想，发现他说的这句话的后半句是不对的，如果你没有将`outfiles`传递给`target`，这些生成的文件根本就不会参与编译连接。之后仔细的看了下`qt5_add_resources`的实现，发现唯一的问题是`outfiles`默认是命名为`qrc_*.cpp`，不过这是个小问题，因为既然是外部资源文件，就一定会被拷贝到可执行文件相对的某个目录或是打包到安装文件中，所以在执行这个操作时重命名就好了。

所以我们这里只要这样执行就可以了：

	qt5_add_resources(outfiles inputfile OPTIONS -binary)

解释下`qt5_add_resources`的实现：

	# qt5_add_resources(outfiles inputfile ... )
	function(QT5_ADD_RESOURCES outfiles )
	    set(options)
	    set(oneValueArgs)
	    set(multiValueArgs OPTIONS)
	
        # 这里不详细解释cmake_parse_arguments的作用，只说明在这个函数内的作用
        # 通过此函数可以获取到所有的qrc文件并放置到_RCC_UNPARSED_ARGUMENTS变量中
        # 另外如果参数中有使用到OPTIONS选项，则将OPTIONS后携带的参数放置到_RCC_OPTIONS变量中
	    cmake_parse_arguments(_RCC "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
	
	    set(rcc_files ${_RCC_UNPARSED_ARGUMENTS})
	    set(rcc_options ${_RCC_OPTIONS})
	
	    foreach(it ${rcc_files})
            # 获取qrc文件的文件名，不包含路径和后缀
	        get_filename_component(outfilename ${it} NAME_WE)
	        get_filename_component(infile ${it} ABSOLUTE)
	        get_filename_component(rc_path ${infile} PATH)
            # **这里就是上面说的那个问题了，默认就生成了cpp文件
	        set(outfile ${CMAKE_CURRENT_BINARY_DIR}/qrc_${outfilename}.cpp)
	
	        set(_RC_DEPENDS)
	        if(EXISTS "${infile}")
	            #  parse file for dependencies
	            #  all files are absolute paths or relative to the location of the qrc file
	            file(READ "${infile}" _RC_FILE_CONTENTS)
                # 匹配后_RC_FILES里的内容为"<file **>**/**/file.**"
	            string(REGEX MATCHALL "<file[^<]+" _RC_FILES "${_RC_FILE_CONTENTS}")
	            foreach(_RC_FILE ${_RC_FILES})
                    # 将<file **>前缀删除，只剩下文件
	                string(REGEX REPLACE "^<file[^>]*>" "" _RC_FILE "${_RC_FILE}")
	                if(NOT IS_ABSOLUTE "${_RC_FILE}")
	                    set(_RC_FILE "${rc_path}/${_RC_FILE}")
	                endif()
                    # 将qrc中的文件设置到_RC_DEPENDS变量中，下面会将此变量添加到add_custom_command的依赖中，
                    # 等文件真实存在时才执行，因为某些情况下，qrc中声明的文件是动态生成的
	                set(_RC_DEPENDS ${_RC_DEPENDS} "${_RC_FILE}")
	            endforeach()
	            # Since this cmake macro is doing the dependency scanning for these files,
	            # let's make a configured file and add it as a dependency so cmake is run
	            # again when dependencies need to be recomputed.
                # 拼凑一个*.qrc.depends到变量out_depends
	            qt5_make_output_file("${infile}" "" "qrc.depends" out_depends)
                # 生成一个和qrc内容一样的*.qrc.depends文件
                # 不过这里没有看懂，因为已经将qrc文件添加到add_custom_command的依赖中了
                # 为什么还要生成另外一个一模一样的文件，并也添加到依赖中？
	            configure_file("${infile}" "${out_depends}" COPYONLY)
	        else()
	            # The .qrc file does not exist (yet). Let's add a dependency and hope
	            # that it will be generated later
	            set(out_depends)
	        endif()
	
            # 执行rcc命令，因为我们添加了-binary参数，所以生成的cpp文件其实是外部的资源文件
	        add_custom_command(OUTPUT ${outfile}
	                           COMMAND ${Qt5Core_RCC_EXECUTABLE}
	                           ARGS ${rcc_options} -name ${outfilename} -o ${outfile} ${infile}
	                           MAIN_DEPENDENCY ${infile}
	                           DEPENDS ${_RC_DEPENDS} "${out_depends}" VERBATIM)
	        list(APPEND ${outfiles} ${outfile})
	    endforeach()
	    set(${outfiles} ${${outfiles}} PARENT_SCOPE)
	endfunction()

### 参考资料
[1] [CMake support for Qt5's external resources](http://anadoxin.org/blog/cmake-support-for-qt5s-external-resources.html)

