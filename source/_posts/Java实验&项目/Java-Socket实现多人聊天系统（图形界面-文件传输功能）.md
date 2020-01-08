---
title: Java Socket实现多人聊天系统（图形界面+文件传输功能）
top: false
toc: true
mathjax: false
abbrlink: 648d85bc
date: 2020-01-08 21:54:25
summary:
tags:
	- Java
	- 聊天系统
categories:
	- Java实验/项目
---

### 前言

GitHub地址：<https://github.com/Samven7/multichat-system>

## 一、多人聊天系统

### 1.1 客户端

Login.java：登录界面

<!-- more -->

~~~java
// Login.java
package exp5;

import java.awt.*;
import javax.swing.*;

public class Login {
	JTextField textField = null;
	JPasswordField pwdField = null;
	ClientReadAndPrint.LoginListen listener=null;
	
	// 构造函数
	public Login() {
		init();
	}
	
	void init() {
		JFrame jf = new JFrame("登录");
		jf.setBounds(500, 250, 310, 210);
		jf.setResizable(false);  // 设置是否缩放
		
		JPanel jp1 = new JPanel();
		JLabel headJLabel = new JLabel("登录界面");
		headJLabel.setFont(new Font(null, 0, 35));  // 设置文本的字体类型、样式 和 大小
		jp1.add(headJLabel);
		
		JPanel jp2 = new JPanel();
		JLabel nameJLabel = new JLabel("用户名：");
		textField = new JTextField(20);
		JLabel pwdJLabel = new JLabel("密码：    ");
		pwdField = new JPasswordField(20);
		JButton loginButton = new JButton("登录");
		JButton registerButton = new JButton("注册");  // 没设置功能
		jp2.add(nameJLabel);
		jp2.add(textField);
		jp2.add(pwdJLabel);
		jp2.add(pwdField);
		jp2.add(loginButton);
		jp2.add(registerButton);
		
		JPanel jp = new JPanel(new BorderLayout());  // BorderLayout布局
		jp.add(jp1, BorderLayout.NORTH);
		jp.add(jp2, BorderLayout.CENTER);
		
		// 设置监控
		listener = new ClientReadAndPrint().new LoginListen();  // 新建监听类
		listener.setJTextField(textField);  // 调用PoliceListen类的方法
		listener.setJPasswordField(pwdField);
		listener.setJFrame(jf);
		pwdField.addActionListener(listener);  // 密码框添加监听
		loginButton.addActionListener(listener);  // 按钮添加监听
		
		jf.add(jp);
		jf.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  // 设置关闭图标作用
		jf.setVisible(true);  // 设置可见
	}
}
~~~

ChatView.java：登录成功后的个人聊天界面

~~~java
// ChatView.java
package exp5;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.File;
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;


public class ChatView {
	String userName;  //由客户端登录时设置
	JTextField text;
	JTextArea textArea;
	ClientReadAndPrint.ChatViewListen listener;
	
	// 构造函数
	public ChatView(String userName) {
		this.userName = userName ;
		init();
	}
	// 初始化函数
	void init() {
		JFrame jf = new JFrame("客户端");
		jf.setBounds(500,200,400,330);  //设置坐标和大小
		jf.setResizable(false);  // 缩放为不能缩放
		
		JPanel jp = new JPanel();
		JLabel lable = new JLabel("用户：" + userName);
		textArea = new JTextArea("***************登录成功，欢迎来到多人聊天室！****************\n",12, 35);
		textArea.setEditable(false);  // 设置为不可修改
		JScrollPane scroll = new JScrollPane(textArea);  // 设置滚动面板（装入textArea）
		scroll.setVerticalScrollBarPolicy(JScrollPane.VERTICAL_SCROLLBAR_ALWAYS);  // 显示垂直条
		jp.add(lable);
		jp.add(scroll);
		
		text = new JTextField(20);
		JButton button = new JButton("发送");
		JButton openFileBtn = new JButton("发送文件");
		jp.add(text);
		jp.add(button);
		jp.add(openFileBtn);
		
		// 设置“打开文件”监听
		openFileBtn.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				showFileOpenDialog(jf);
			}
		});
		
		// 设置“发送”监听
		listener = new ClientReadAndPrint().new ChatViewListen();
		listener.setJTextField(text);  // 调用PoliceListen类的方法
		listener.setJTextArea(textArea);
		listener.setChatViewJf(jf);
		text.addActionListener(listener);  // 文本框添加监听
		button.addActionListener(listener);  // 按钮添加监听
		
		jf.add(jp);
		jf.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  // 设置右上角关闭图标的作用
		jf.setVisible(true);  // 设置可见
	}
	// “打开文件”调用函数
	void showFileOpenDialog(JFrame parent) {
		// 创建一个默认的文件选择器
		JFileChooser fileChooser = new JFileChooser();
		// 设置默认显示的文件夹
		fileChooser.setCurrentDirectory(new File("C:/Users/Samven/Desktop"));
		// 添加可用的文件过滤器（FileNameExtensionFilter 的第一个参数是描述, 后面是需要过滤的文件扩展名）
//        fileChooser.addChoosableFileFilter(new FileNameExtensionFilter("(txt)", "txt"));
        // 设置默认使用的文件过滤器（FileNameExtensionFilter 的第一个参数是描述, 后面是需要过滤的文件扩展名 可变参数）
        fileChooser.setFileFilter(new FileNameExtensionFilter("(txt)", "txt"));
		// 打开文件选择框（线程将被堵塞，知道选择框被关闭）
		int result = fileChooser.showOpenDialog(parent);  // 对话框将会尽量显示在靠近 parent 的中心
		// 点击确定
		if(result == JFileChooser.APPROVE_OPTION) {
			// 获取路径
			File file = fileChooser.getSelectedFile();
			String path = file.getAbsolutePath();
			ClientFileThread.outFileToServer(path);
		}
	}
}
~~~

Client.java：客户端

~~~java
// Client.java
package exp5;

import java.net.*;
import javax.swing.*;
import java.awt.event.*;
import java.io.*;

public class Client {
	// 主函数，新建登录窗口
	public static void main(String[] args) {
		new Login();
	}
}

/**
 *  负责客户端的读和写，以及登录和发送的监听
 *  之所以把登录和发送的监听放在这里，是因为要共享一些数据，比如mySocket,textArea
 */
class ClientReadAndPrint extends Thread{
	static Socket mySocket = null;  // 一定要加上static，否则新建线程时会清空
	static JTextField textInput;
	static JTextArea textShow;
	static JFrame chatViewJFrame;
	static BufferedReader in = null;
	static PrintWriter out = null;
	static String userName;
	
	// 用于接收从服务端发送来的消息
	public void run() {
		try {
			in = new BufferedReader(new InputStreamReader(mySocket.getInputStream()));  // 输入流
			while (true) {
				String str = in.readLine();  // 获取服务端发送的信息
				textShow.append(str + '\n');  // 添加进聊天客户端的文本区域
				textShow.setCaretPosition(textShow.getDocument().getLength());  // 设置滚动条在最下面
			}
		} catch (Exception e) {}
	}
	
	/**********************登录监听(内部类)**********************/
	class LoginListen implements ActionListener{
		JTextField textField;
		JPasswordField pwdField;
		JFrame loginJFrame;  // 登录窗口本身
		
		ChatView chatView = null;
		
		public void setJTextField(JTextField textField) {
			this.textField = textField;
		}
		public void setJPasswordField(JPasswordField pwdField) {
			this.pwdField = pwdField;
		}
		public void setJFrame(JFrame jFrame) {
			this.loginJFrame = jFrame;
		}
		public void actionPerformed(ActionEvent event) {
			userName = textField.getText();
			String userPwd = String.valueOf(pwdField.getPassword());  // getPassword方法获得char数组
			if(userName.length() >= 1 && userPwd.equals("123")) {  // 密码为123并且用户名长度大于等于1
				chatView = new ChatView(userName);  // 新建聊天窗口,设置聊天窗口的用户名（静态）
				// 建立和服务器的联系
				try {
					InetAddress addr = InetAddress.getByName(null);  // 获取主机地址
					mySocket = new Socket(addr,8081);  // 客户端套接字
					loginJFrame.setVisible(false);  // 隐藏登录窗口
					out = new PrintWriter(mySocket.getOutputStream());  // 输出流
					out.println("用户【" + userName + "】进入聊天室！");  // 发送用户名给服务器
					out.flush();  // 清空缓冲区out中的数据
				} catch (IOException e) {
					e.printStackTrace();
				}
				// 新建普通读写线程并启动
				ClientReadAndPrint readAndPrint = new ClientReadAndPrint();
				readAndPrint.start();
				// 新建文件读写线程并启动
				ClientFileThread fileThread = new ClientFileThread(userName, chatViewJFrame, out);
				fileThread.start();
			}
			else {
				JOptionPane.showMessageDialog(loginJFrame, "账号或密码错误，请重新输入！", "提示", JOptionPane.WARNING_MESSAGE);
			}
		}
	}
	
	/**********************聊天界面监听(内部类)**********************/
	class ChatViewListen implements ActionListener{
		public void setJTextField(JTextField text) {
			textInput = text;  // 放在外部类，因为其它地方也要用到
		}
		public void setJTextArea(JTextArea textArea) {
			textShow = textArea;  // 放在外部类，因为其它地方也要用到
		}
		public void setChatViewJf(JFrame jFrame) {
			chatViewJFrame = jFrame;  // 放在外部类，因为其它地方也要用到
			// 设置关闭聊天界面的监听
			chatViewJFrame.addWindowListener(new WindowAdapter() {
				public void windowClosing(WindowEvent e) {
					out.println("用户【" + userName + "】离开聊天室！");
					out.flush();
					System.exit(0);
				}
			});
		}
		// 监听执行函数
		public void actionPerformed(ActionEvent event) {
			try {
				String str = textInput.getText();
				// 文本框内容为空
				if("".equals(str)) {
					textInput.grabFocus();  // 设置焦点（可行）
					// 弹出消息对话框（警告消息）
					JOptionPane.showMessageDialog(chatViewJFrame, "输入为空，请重新输入！", "提示", JOptionPane.WARNING_MESSAGE);
					return;
				}
				out.println(userName + "说：" + str);  // 输出给服务端
				out.flush();  // 清空缓冲区out中的数据
				
				textInput.setText("");  // 清空文本框
				textInput.grabFocus();  // 设置焦点（可行）
//				textInput.requestFocus(true);  // 设置焦点（可行）
			} catch (Exception e) {}
		}
	}
}
~~~

ClientFileThread.java：文件传输功能（客户端）

~~~java
// ClientFileThread.java
package exp5;

import java.io.*;
import java.net.*;
import javax.swing.*;

public class ClientFileThread extends Thread{
	private Socket socket = null;
	private JFrame chatViewJFrame = null;
	static String userName = null;
	static PrintWriter out = null;  // 普通消息的发送（Server.java传来的值）
	static DataInputStream fileIn = null;
	static DataOutputStream fileOut = null;
	static DataInputStream fileReader = null;
	static DataOutputStream fileWriter = null;
	
	public ClientFileThread(String userName, JFrame chatViewJFrame, PrintWriter out) {
		ClientFileThread.userName = userName;
		this.chatViewJFrame = chatViewJFrame;
		ClientFileThread.out = out;
	}
	
	// 客户端接收文件
	public void run() {
		try {
			InetAddress addr = InetAddress.getByName(null);  // 获取主机地址
			socket = new Socket(addr, 8090);  // 客户端套接字
			fileIn = new DataInputStream(socket.getInputStream());  // 输入流
			fileOut = new DataOutputStream(socket.getOutputStream());  // 输出流
			// 接收文件
			while(true) {
				String textName = fileIn.readUTF();
				long totleLength = fileIn.readLong();
				int result = JOptionPane.showConfirmDialog(chatViewJFrame, "是否接受？", "提示",
														   JOptionPane.YES_NO_OPTION);
				int length = -1;
				byte[] buff = new byte[1024];
				long curLength = 0;
				// 提示框选择结果，0为确定，1位取消
				if(result == 0){
//					out.println("【" + userName + "选择了接收文件！】");
//					out.flush();
					File userFile = new File("C:\\Users\\Samven\\Desktop\\接受文件\\" + userName);
					if(!userFile.exists()) {  // 新建当前用户的文件夹
						userFile.mkdir();
					}
					File file = new File("C:\\Users\\Samven\\Desktop\\接受文件\\" + userName + "\\"+ textName);
					fileWriter = new DataOutputStream(new FileOutputStream(file));
					while((length = fileIn.read(buff)) > 0) {  // 把文件写进本地
						fileWriter.write(buff, 0, length);
						fileWriter.flush();
						curLength += length;
//						out.println("【接收进度:" + curLength/totleLength*100 + "%】");
//						out.flush();
						if(curLength == totleLength) {  // 强制结束
							break;
						}
					}
					out.println("【" + userName + "接收了文件！】");
					out.flush();
					// 提示文件存放地址
					JOptionPane.showMessageDialog(chatViewJFrame, "文件存放地址：\n" +
							"C:\\Users\\Samven\\Desktop\\接受文件\\" +
							userName + "\\" + textName, "提示", JOptionPane.INFORMATION_MESSAGE);
				}
				else {  // 不接受文件
					while((length = fileIn.read(buff)) > 0) {
						curLength += length;
						if(curLength == totleLength) {  // 强制结束
							break;
						}
					}
				}
				fileWriter.close();
			}
		} catch (Exception e) {}
	}
	
	// 客户端发送文件
	static void outFileToServer(String path) {
		try {
			File file = new File(path);
			fileReader = new DataInputStream(new FileInputStream(file));
			fileOut.writeUTF(file.getName());  // 发送文件名字
			fileOut.flush();
			fileOut.writeLong(file.length());  // 发送文件长度
			fileOut.flush();
			int length = -1;
			byte[] buff = new byte[1024];
			while ((length = fileReader.read(buff)) > 0) {  // 发送内容
				
				fileOut.write(buff, 0, length);
				fileOut.flush();
			}
			
			out.println("【" + userName + "已成功发送文件！】");
			out.flush();
		} catch (Exception e) {}
	}
}
~~~



### 1.2 服务器端

MultiChat.java：多人聊天系统界面（服务器端）

~~~java
// MultiChat.java
package exp5;

import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

import javax.swing.*;

public class MultiChat {
	JTextArea textArea;
	
	// 用于向文本区域添加信息
	void setTextArea(String str) {
		textArea.append(str+'\n');
		textArea.setCaretPosition(textArea.getDocument().getLength());  // 设置滚动条在最下面
	}
	
	// 构造函数
	public MultiChat() {
		init();
	}
	
	void init() {
		JFrame jf = new JFrame("服务器端");
		jf.setBounds(500,100,450,500);  // 设置窗口坐标和大小
		jf.setResizable(false);  // 设置为不可缩放
		
		JPanel jp = new JPanel();  // 新建容器
		JLabel lable = new JLabel("==欢迎来到多人聊天系统（服务器端）==");
		textArea = new JTextArea(23, 38);  // 新建文本区域并设置长宽
		textArea.setEditable(false);  // 设置为不可修改
		JScrollPane scroll = new JScrollPane(textArea);  // 设置滚动面板（装入textArea）
		scroll.setVerticalScrollBarPolicy(ScrollPaneConstants.VERTICAL_SCROLLBAR_ALWAYS);  // 显示垂直条
		jp.add(lable);
		jp.add(scroll);
		
		jf.addWindowListener(new WindowAdapter() {
			public void windowClosing(WindowEvent e) {
				System.exit(0);
			}
		});
		
		jf.add(jp);
		jf.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  // 设置关闭图标作用
		jf.setVisible(true);  // 设置可见
	}
}

~~~

Server.java：服务器端

~~~java
// Server.java
package exp5;

import java.io.*;
import java.net.*;
import java.util.*;

public class Server{
	static ServerSocket server = null;
	static Socket socket = null;
	static List<Socket> list = new ArrayList<Socket>();  // 存储客户端
	
	public static void main(String[] args) {
		MultiChat multiChat = new MultiChat();  // 新建聊天系统界面
		try {
			// 在服务器端对客户端开启文件传输的线程
			ServerFileThread serverFileThread = new ServerFileThread();
			serverFileThread.start();
			server = new ServerSocket(8081);  // 服务器端套接字（只能建立一次）
			// 等待连接并开启相应线程
			while (true) {
				socket = server.accept();  // 等待连接
				list.add(socket);  // 添加当前客户端到列表
				// 在服务器端对客户端开启相应的线程
				ServerReadAndPrint readAndPrint = new ServerReadAndPrint(socket, multiChat);
				readAndPrint.start();
			}
		} catch (IOException e1) {
			e1.printStackTrace();  // 出现异常则打印出异常的位置
		}
	}
}

/**
 *  服务器端读写类线程
 *  用于服务器端读取客户端的信息，并把信息发送给所有客户端
 */
class ServerReadAndPrint extends Thread{
	Socket nowSocket = null;
	MultiChat multiChat = null;
	BufferedReader in =null;
	PrintWriter out = null;
	// 构造函数
	public ServerReadAndPrint(Socket s, MultiChat multiChat) {
		this.multiChat = multiChat;  // 获取多人聊天系统界面
		this.nowSocket = s;  // 获取当前客户端
	}
	
	public void run() {
		try {
			in = new BufferedReader(new InputStreamReader(nowSocket.getInputStream()));  // 输入流
			// 获取客户端信息并把信息发送给所有客户端
			while (true) {
				String str = in.readLine();
				// 发送给所有客户端
				for(Socket socket: Server.list) {
					out = new PrintWriter(socket.getOutputStream());  // 对每个客户端新建相应的socket套接字
					if(socket == nowSocket) {  // 发送给当前客户端
						out.println("(你)" + str);
					}
					else {  // 发送给其它客户端
						out.println(str);
					}
					out.flush();  // 清空out中的缓存
				}
				// 调用自定义函数输出到图形界面
				multiChat.setTextArea(str);
			}
		} catch (Exception e) {
			Server.list.remove(nowSocket);  // 线程关闭，移除相应套接字
		}
	}
}
~~~

ServerFileThread.java：文件传输功能（服务器端）

~~~java
// ServerFileThread.java
package exp5;

import java.io.*;
import java.net.*;
import java.util.ArrayList;
import java.util.List;

public class ServerFileThread extends Thread{
	ServerSocket server = null;
	Socket socket = null;
	static List<Socket> list = new ArrayList<Socket>();  // 存储客户端
	
	public void run() {
		try {
			server = new ServerSocket(8090);
			while(true) {
				socket = server.accept();
				list.add(socket);
				// 开启文件传输线程
				FileReadAndWrite fileReadAndWrite = new FileReadAndWrite(socket);
				fileReadAndWrite.start();
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

class FileReadAndWrite extends Thread {
	private Socket nowSocket = null;
	private DataInputStream input = null;
	private DataOutputStream output = null;
	
	public FileReadAndWrite(Socket socket) {
		this.nowSocket = socket;
	}
	public void run() {
		try {
			input = new DataInputStream(nowSocket.getInputStream());  // 输入流
			while (true) {
				// 获取文件名字和文件长度
				String textName = input.readUTF();
				long textLength = input.readLong();
				// 发送文件名字和文件长度给所有客户端
				for(Socket socket: ServerFileThread.list) {
					output = new DataOutputStream(socket.getOutputStream());  // 输出流
					if(socket != nowSocket) {  // 发送给其它客户端
						output.writeUTF(textName);
						output.flush();
						output.writeLong(textLength);
						output.flush();
					}
				}
				// 发送文件内容
				int length = -1;
				long curLength = 0;
				byte[] buff = new byte[1024];
				while ((length = input.read(buff)) > 0) {
					curLength += length;
					for(Socket socket: ServerFileThread.list) {
						output = new DataOutputStream(socket.getOutputStream());  // 输出流
						if(socket != nowSocket) {  // 发送给其它客户端
							output.write(buff, 0, length);
							output.flush();
						}
					}
					if(curLength == textLength) {  // 强制退出
						break;
					}
				}
			}
		} catch (Exception e) {
			ServerFileThread.list.remove(nowSocket);  // 线程关闭，移除相应套接字
		}
	}
}
~~~



## 二、运行效果

### 2.1 初始化
- 服务器端（先运行Server.java）
![Snipaste_2019-10-14_21-07-09](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmE0LnNpbmFpbWcuY24vbGFyZ2UvMDA2VnJKQUpseTFnN3kxYW5sc2ZnajMwYngwZG9qcmUuanBn?x-oss-process=image/format,png)
- 登录界面（接着运行Client.java，运行一次生成一个登录界面）
![Snipaste_2019-10-14_21-08-44](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmEyLnNpbmFpbWcuY24vbGFyZ2UvMDA2VnJKQUpseTFnN3kxYncwbHp4ajMwaGkwYzZteGsuanBn?x-oss-process=image/format,png)


### 2.2 登录成功
![Snipaste_2019-10-14_21-12-23](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmF4NC5zaW5haW1nLmNuL2xhcmdlLzAwNlZySkFKbHkxZzd5MWVjdWRkMGozMG1tMGVpZGdhLmpwZw?x-oss-process=image/format,png)
![Snipaste_2019-10-14_21-13-03](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmExLnNpbmFpbWcuY24vbGFyZ2UvMDA2VnJKQUpseTFnN3kxZjBudGMzajMwY3owZWZ0OHQuanBn?x-oss-process=image/format,png)
### 2.3 发送信息
![Snipaste_2019-10-14_22-01-35](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmF4NC5zaW5haW1nLmNuL2xhcmdlLzAwNlZySkFKZ3kxZzd5MnRqaGJwcmozMHVxMGR5d2ZiLmpwZw?x-oss-process=image/format,png)
### 2.4 发送文件
打开文件我设置了默认路径是在桌面。接受文件需要先在桌面创建一个名为“接受文件”的文件夹，用于存放所有用户接收的文件。
![Snipaste_2019-10-14_22-05-00](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90dmE0LnNpbmFpbWcuY24vbGFyZ2UvMDA2VnJKQUpneTFnN3kyeDNzbWNpajMwbGkwZWtteTYuanBn?x-oss-process=image/format,png)