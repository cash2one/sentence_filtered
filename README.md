<pre>
sentence_filtered
语言模型过滤不匹配句子，通过fabric进行远程控制

sentence.txt（清洗后的原始数据）==>  *.prebuild ==> *.packet(*.packet文件直
	接交由horde进行词频抓取)
	（脚本路径：/home/ferrero/cloudinn/filtered_unmatch_sentence）
	1、sentence.txt ==> *.packet（两个方法，1)只保留汉字，2)保留汉字+数字+字母+空格）
	（USAGE: -i sentence_filename）
		1、sentence2Packet.py	
			标音中的分词逻辑:在非汉字部分进行切割，即不存在数字、字母、空格的情况
		2、sentence2Packet_with_num_letter.py
			标音中的分词逻辑:在非汉字/字母/数字/空格处进行切割
				1.若line中只有"字母+数字"，则保留期间空格
				2.若line中"汉字"，则将line中的空格置空
	2、sentence.txt ==> *.prebuild
		(USAGE: -s sentence_filename -d prebuild_filename)#sentence_filename为绝对路径，
		若有多列(\t隔开)，则视最后一列为词频被写入到*.prebuild文件中。
		1、sentence2Prebuild.py
			标音中的分词逻辑:在非汉字部分进行切割，即不存在数字、字母、空格的情况
		2、sentence2Prebuild_with_num_letter.py
			标音中的分词逻辑:在非汉字/字母/数字/空格处进行切割
				1.若line中只有"字母+数字"，则保留期间空格
				2.若line中"汉字"，则将line中的空格置空		
***********************************************************************************
若需要使用fabric时通过subprocess中的call方法执行fab_command且不添加-f参数，
	则需要在目录中添加fabfile.py文件，
文件中可以设置env的一些参数，例如（env.use_ssh_config = True）   

fab [options] -- [shell command]  

所有在 '--'之后的语句将会转化为run()来执行  
command = 'cd /home/wanghuafeng/cloud_word/node-sri/test/unmatch_ngram_filter; ls'  
os.system('fab -H unicorn -- "%s"'%command)
	#在unicorn上执行command语句（注意，此处双引号不能缺少）    
os.system('fab -H "unicorn, s2, s3, ana" -- "uname -a"')
	#在远程服务器unicorn, s2, s3, ana上执行 uname -a 语句    
设置host的方式：   
	* Per-task, command-line host lists (fab mytask:host=host1) 
		override absolutely everything else.   
	* Per-task, decorator-specified host lists (@hosts('host1')) override the env variables.   
	* Globally specified host lists set in the fabfile (env.hosts = ['host1']) can override
	  such lists set on the command-line, but only if you’re not careful (or want them to.)    
	* Globally specified host lists set on the command-line
		(--hosts=host1) will initialize the envvariables, but that’s it.        

fab_command = '''fab -H unicorn -- 
				"cd /home/wanghuafeng/cloud_word/node-sri/test/unmatch_ngram_filter;
				python split_file.py -f ghost.packet -c 10;
				mkdir splited_data;
				mv ghost.packet.partial_* splited_data"'''
exec_fab文件将filtered_sentence.py拷贝到远程s3服务器中的指定目录，
并在s3执行该文件，同时将标准输出显示在本地以便调试

关于阻塞与非阻塞子进程使用：  
（1）当父进程不需要获取子进程数据，或者某子进程的操作比较耗时。应该使用非阻塞式，此时即
	便该子进程运行过程中crash同样不会影响父进程的正常运行，也即，当父进程fork出该子进程
	以后，他们没有了什么关系。（特殊情况下，通过fabric远程执行操作时，父进程在执行结束后
	用户便退出，fabric连接断开。但是非阻塞式执行的子进程却可能还没有结束，而在用户退出情
	况下，该用户执行的所有的进程都会被kill掉。故这种情况下，通过非阻塞式生成的子进程程序
	必须使用nohup command &来保证当父进程结束切用户退出时，该子进程依旧能够正常进行）
非阻塞式:subprocess.Popen(command, shell=True)
	耗时较久的操作，此处为非阻塞式子进程运行，不会影响常规数据流程    
（2）当父进程与子进程有数据交互（进程通讯），或者子进程crash时要求父进程同样
	中断，此时比较适合阻塞式    
阻塞式:subprocess.call(command, shell=True)    
	而实际上:subprocess.call(*popenargs, **kwargs) 
	即为 subprocess.Popen(*popenargs, **kwargs).wait()进行了已成封装    

另：stdout.read()的数据总是为ASCII（使用popen.stdout.readlines()时可逐行进行decode('utf-8')）    
</pre>