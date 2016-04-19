title: .wav声音文件的二三事
date: 2015-04-21 19:30:15
tags: Pick Up
---
前一段时间，因为实验室项目的需要，我学习了一下.wav声音文件的一些东西，总结如下：

首先，参考一篇对.wav文件头的介绍，明白了.wav文件的构成实质上是由一部分控制文件实际信息的文件头与真正的语音数据组成。参考文章链接戳[这里](http://wenku.baidu.com/link?url=X2D9QlP503m8i7jJp-NAC76sY0TrbSxqpkUQxTQttUj3mEDW3lEc5SbDEaJnhlv-PJU_UY0oekvqanM1YjKcL1W4i6V4Ws5nhtwUSaefDbS)。
在实验室的项目中，我们获得了一定大小的解码文件，在获取到解码文件后，希望能够在硬盘中保存为.wav的音频文件，供今后的播放使用。结合参考文献，我编写了一个将采样率为8000Hz、量化位数为16比特、通道数为1的声音的纯数据文件打包成可以用播放器直接播放的.wav文件。供参考：
<!-- more -->
``` bash
/*
@author:Xu Ruiqiang
 
函数实现根据获取的解码文件来构造合适的.wav的文件头。关键是在文件头44字节的04H开始与28H开始处。
示例：
-----------------------------------------------------------------------------
|  Decoding File Size  |  .wav File Size    |	f_size	 |		b_size		|
|     166080 Bytes     |	166124 Bytes	|166116 Bytes|	166080 Bytes	|
-----------------------------------------------------------------------------
将获取的f_size与b_size转换为16进制，然后按照little-endian的原则构造即可
例如:
f_size为166116，变成16进制为000288E4，那么后边的那几个字节应当为 E4 88 02 00
b_size为166080，变成16进制为000288C0，那么后边的那几个字节应当为 C0 88 02 00
*/
 
#include<fstream>
#include<iostream>
#include<io.h>
using namespace std;
void main()
{
	/*
	ifstream input;
	input.open("t_8k_2s.wav",ios::binary | ios::in);		//二进制方式打开 | 文件以输入方式打开
	if(input==NULL)
	{
	cout<<"opening the original sound file failed"<<endl;
	return;
	}
	char buffer[50];
	input.read(buffer,44);		//获取原来声音文件的前44个字节，并且保存在buffer中
	int l1=input.gcount();		//函数gcount用来获取实际读取的字符数
	cout<<l1<<endl;
	input.close();
	ofstream output;
	output.open("out.wav",ios::out | ios::trunc| ios::binary);		//文件以输出方式打开 | 如果文件存在，则文件长度为0 | 以二进制方式打开文件
	if(output==NULL)
	{
	cout<<"open out failded"<<endl;
	return;
	}
	output.write(buffer,44);		//将已经获取的文件头写到out.wav中
	ifstream input2;
	input2.open("-l",ios::binary | ios::in);			//以二进制方式打开文件 | 文件以输入方式打开
	char buf[1024];
	int len=0;
	input2.read(buf,1024);		//读取1024字节
	len=input2.gcount();
	//循环读取的过程，每次均读取1024个字节，直到读取结束（len = 0）
	while(len!=0)
	{
	output.write(buf,len);		//写入1024字节到文件中
	input2.read(buf,1024);		//继续读取1024字节
	len=input2.gcount();
	}
	input2.close();
	output.close();
	*/
	void Dec2Hex(long size, char result[]);
	int Convert(char a, char b);
	int Hex2Dec(char a);
 
 
	//codes written by XRQ
	int handle,header4,header5,header6,header7,header40,header41,header42,header43;		//8 sensitive variable concerning the size of the .wav file
	long size,fullsize,f_size,b_size;		//f_size and b_size are Bytes at offset 04H and 28H
	char f_sizeHex[8], b_sizeHex[8];
 
 
	//get the size of the decoding file
	handle = open("-l",0x0100);
	size = filelength(handle);		//the size of the decoding files in Bytes	
	if(size == 0)
	{
		cout<<"open files when trying to get file size failed"<<endl;
		return;
	}
	close(handle);
 
 
	fullsize = size + 44;		//finally the size of the .wav size should be 44 bytes bigger than the decoding file.
	f_size = fullsize - 8;		//Bytes at offset 04H
	b_size = fullsize - 44;		//Bytes at offset 28H
	Dec2Hex(f_size,f_sizeHex);		//get the f_size of the .wav in Hex.
	Dec2Hex(b_size,b_sizeHex);		//get the b_size of the .wav in Hex.
 
 
	//Get the most confusing 8 Bytes in the 44 Bytes according to the file size
	header4 = Convert(f_sizeHex[6], f_sizeHex[7]);
	header5 = Convert(f_sizeHex[4], f_sizeHex[5]);
	header6 = Convert(f_sizeHex[2], f_sizeHex[3]);
	header7 = Convert(f_sizeHex[0], f_sizeHex[1]);
	header40 = Convert(b_sizeHex[6], b_sizeHex[7]);
	header41 = Convert(b_sizeHex[4], b_sizeHex[5]);
	header42 = Convert(b_sizeHex[2], b_sizeHex[3]);
	header43 = Convert(b_sizeHex[0], b_sizeHex[1]);
	cout<<header4<<" "<<header5<<" "<<header6<<" "<<header7<<endl;
	cout<<header40<<" "<<header41<<" "<<header42<<" "<<header43<<endl;
 
 
	//formating the first 44 Bytes according to the file info.
	char header[44] = {
		char(82),char(73),char(70),char(70),char(header4),char(header5),char(header6),char(header7),char(87),char(65),char(86),
		char(69),char(102),char(109),char(116),char(32),char(16),char(0),char(0),char(0),char(1),char(0),
		char(1),char(0),char(64),char(31),char(0),char(0),char(-128),char(62),char(0),char(0),char(2),
		char(0),char(16),char(0),char(100),char(97),char(116),char(97),char(header40),char(header41),char(header42),char(header43)};
 
 
	ofstream output;
	output.open("out.wav", ios::out | ios::trunc | ios::binary);		//open the file in binary
	if( output == NULL )
	{
		cout<<"Opening output failed"<<endl;
		return;
	}
	output.write(header,44);		//copy the .wav file header
 
 
	//copy the decoding data to the output file.
	ifstream input;
	input.open("-l",ios::binary | ios::in);
	char buf[1024];
	int len = 0;
	input.read(buf,1024);
	len = input.gcount();
	while( len != 0 )
	{
		output.write(buf,len);
		input.read(buf,1024);
		len = input.gcount();
	}
	input.close();
	output.close();
}
 
 
void Dec2Hex(long size, char result[])		//convert a decimal number to a hex number as a char array, 4 Bytes(8 elements) maximum.
{
	int a[8]={0,0,0,0,0,0,0,0}, remainder, i = 0, j = 0;
	char hex[16] = {'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
	while(size > 0)
	{
		remainder = size % 16;
		a[i++] = remainder;
		size = size / 16;
	}
	for (; j < 8; j++)
	{
		result[j] = hex[a[7 - j]];
	}
	return;
}
int Hex2Dec(char a)		//conver a single hex char to a binary. eg: Hex('E') = 14
{
	int result;
	switch(a) {
	case '0': result = 0; break;
	case '1': result = 1; break;
	case '2': result = 2; break;
	case '3': result = 3; break;
	case '4': result = 4; break;
	case '5': result = 5; break;
	case '6': result = 6; break;
	case '7': result = 7; break;
	case '8': result = 8; break;
	case '9': result = 9; break;
	case 'A': result = 10; break;
	case 'B': result = 11; break;
	case 'C': result = 12; break;
	case 'D': result = 13; break;
	case 'E': result = 14; break;
	case 'F': result = 15; break;
	}
	return result;
}
int Convert(char a, char b)		//combining 2 hex char and conver to a decimal. eg:Conver('7','D')=125
{
	int a_dec, b_dec;
	a_dec = Hex2Dec(a);
	b_dec = Hex2Dec(b);
	return a_dec * 16 + b_dec;
 
}
```
以上代码为C++代码，我特地移植到了C语言上：
``` bash
void waveformating()
{
	//codes written by XRQ
	int len, i = 0, handle = 0,header4 = 0,header5 = 0,header6 = 0,header7 = 0,header40 = 0,header41 = 0,header42 = 0,header43 = 0;		//8 sensitive variable concerning the size of the .wav file
	long size,fullsize,f_size,b_size;		//f_size and b_size are Bytes at offset 04H and 28H
	char f_sizeHex[8], b_sizeHex[8],buffer[1024];
	FILE * pFile, * qFile;
 
	//the .wav sound file head concerning the sound file info.
	char header[44] = {
		(char)82,(char)73,(char)70,(char)70,(char)header4,(char)header5,(char)header6,(char)header7,(char)87,(char)65,(char)86,
		(char)69,(char)102,(char)109,(char)116,(char)32,(char)16,(char)0,(char)0,(char)0,(char)1,(char)0,
		(char)1,(char)0,(char)64,(char)31,(char)0,(char)0,(char)-128,(char)62,(char)0,(char)0,(char)2,
		(char)0,(char)16,(char)0,(char)100,(char)97,(char)116,(char)97,(char)header40,(char)header41,(char)header42,(char)header43};
 
 
	//get the size of the decoding file. Attention: make sure the previous file stream has been closed.
	pFile = fopen("out","rb");
	if (pFile == NULL)
	{
		printf("open files when trying to get file size failed\n");
	}
	else
	{
		fseek(pFile,0,SEEK_END);
		size = ftell(pFile);		//the size of the decoding file in Bytes
		fclose(pFile);
	}
 
	fullsize = size + 44;		//finally the size of the .wav size should be 44 bytes bigger than the decoding file.
	f_size = fullsize - 8;		//Bytes at offset 04H
	b_size = fullsize - 44;		//Bytes at offset 28H
	Dec2Hex(f_size,f_sizeHex);		//get the f_size of the .wav in Hex.
	Dec2Hex(b_size,b_sizeHex);		//get the b_size of the .wav in Hex.
 
 
	//Get the most confusing 8 Bytes in the 44 Bytes according to the file size
	header4 = Convert(f_sizeHex[6], f_sizeHex[7]);
	header5 = Convert(f_sizeHex[4], f_sizeHex[5]);
	header6 = Convert(f_sizeHex[2], f_sizeHex[3]);
	header7 = Convert(f_sizeHex[0], f_sizeHex[1]);
	header40 = Convert(b_sizeHex[6], b_sizeHex[7]);
	header41 = Convert(b_sizeHex[4], b_sizeHex[5]);
	header42 = Convert(b_sizeHex[2], b_sizeHex[3]);
	header43 = Convert(b_sizeHex[0], b_sizeHex[1]);
	header[4] = (char)header4;
	header[5] = (char)header5;
	header[6] = (char)header6;
	header[7] = (char)header7;
	header[40] = (char)header40;
	header[41] = (char)header41;
	header[42] = (char)header42;
	header[43] = (char)header43;
 
	//Here starts the .wav sound file formating
	pFile = fopen("out.wav","wb+");
	if(pFile == NULL)
	{
		printf("opening out.wav failed.\n");
		return;
	}
	len = fwrite(header,1,44,pFile);	//copy the .wav sound file header(44 Bytes)
 
 
	//The following progress is about copying the decoding data to the output file
	qFile = fopen("out","rb");
	len = fread(buffer,1,1024,qFile);
	while(len != 0)
	{
		len = fwrite(buffer,1,1024,pFile);	//This assignment to the variable 'len' is not a must.
		len = fread(buffer,1,1024,qFile);
		i++;
	}
 
	fclose(pFile);
	fclose(qFile);
}
 
void Dec2Hex(long size, char result[])		//convert a decimal number to a hex number as a char array, 4 Bytes(8 elements) maximum.
{
	int a[8]={0,0,0,0,0,0,0,0}, remainder, i = 0, j = 0;
	char hex[16] = {'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
	while(size > 0)
	{
		remainder = size % 16;
		a[i++] = remainder;
		size = size / 16;
	}
	for (; j < 8; j++)
	{
		result[j] = hex[a[7 - j]];
	}
	return;
}
int Hex2Dec(char a)		//conver a single hex char to a binary. eg: Hex('E') = 14
{
	int result;
	switch(a) {
	case '0': result = 0; break;
	case '1': result = 1; break;
	case '2': result = 2; break;
	case '3': result = 3; break;
	case '4': result = 4; break;
	case '5': result = 5; break;
	case '6': result = 6; break;
	case '7': result = 7; break;
	case '8': result = 8; break;
	case '9': result = 9; break;
	case 'A': result = 10; break;
	case 'B': result = 11; break;
	case 'C': result = 12; break;
	case 'D': result = 13; break;
	case 'E': result = 14; break;
	case 'F': result = 15; break;
	}
	return result;
}
int Convert(char a, char b)		//combining 2 hex char and conver to a decimal. eg:Conver('7','D')=125
{
	int a_dec, b_dec;
	a_dec = Hex2Dec(a);
	b_dec = Hex2Dec(b);
	return a_dec * 16 + b_dec;
}
```