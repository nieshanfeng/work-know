## netty
### NIO基础
#### BIO 同步阻塞式I/O
```
#####################TimeServer ####################
package com.phei.netty.bio;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class TimeServer {

    public static void main(String[] args) throws IOException {
        int port = 8080;
        if (args != null && args.length > 0) {
            try {
                port = Integer.valueOf(args[0]);
            } catch (NumberFormatException e){
            }
        }

        ServerSocket server = null;
        try {
            server = new ServerSocket(port);
            System.out.println("The time server is start in port:" + port);
            Socket socket = null;
            while (true) {
                socket = server.accept();
                new Thread(new TimeServerHandler(socket)).start();

            }
        } finally {
            if (server != null) {
                System.out.println("The time server close");
                server.close();
                server = null;
            }
        }
    }
}

#####################TimeServerHandler ####################
package com.phei.netty.bio;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.Date;

public class TimeServerHandler implements Runnable {
    private  Socket socket;
    public TimeServerHandler(Socket socket) {
        this.socket=socket;
    }

    public void run() {
        BufferedReader in=null;
        PrintWriter out=null;
        try {
            in=new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
            out =new PrintWriter(this.socket.getOutputStream(),true);
            String currentTime=null;
            String body=null;
            while (true){
                body=in.readLine();
                if(body==null){
                    break;
                }
                System.out.println("The time server receive order :"+body);
                currentTime="QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date( System.currentTimeMillis()).toString() :
                        "BAD ORDER";
                System.out.println(currentTime);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(in !=null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            if(out !=null){
                try {
                    out.close();
                    out=null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            if(this.socket !=null){
                try {
                    this.socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

#####################TimeClient ####################
package com.phei.netty.bio;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class TimeClient {
    public static void main(String[] args) {
        int port = 8080;
        if (args != null && args.length > 0) {
            try {
                port = Integer.valueOf(args[0]);
            } catch (NumberFormatException e){
            }
        }
        Socket socket=null;
        BufferedReader in =null;
        PrintWriter out=null;
        try {
            socket =new Socket("127.0.0.1",port);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out=new PrintWriter(socket.getOutputStream(),true);
            out.println("QUERY TIME ORDER");
            System.out.println("Send order 2 server success.");
            String resp=in.readLine();
            System.out.println("Now is :"+resp);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(out !=null){
                try {
                    out.close();
                    out=null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if(in !=null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(socket !=null){
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

#### NI/O 非阻塞I/O
  - 与Socket类和ServerSocket类相对应，NIO也提供了SocketChannel和ServerSocketChannel两种不同的
  套接字通道实现。这两种新增的通道都支持阻塞和非阻塞两种模式
  - Channnel是一个通道，可以通过它读取和写入数据，它就像自来水管一样，网络数据通过Channel读取和
  写入，通道与流的不同之处在于通道是双向的，流只是在一个方向上移动(一个流必须是InputStream或者OutputStream的子类),
  而且通道可以用于读、写或者同时用于读写
  - Channel可以分为两大类:分别是用于网络读写的SelectableChannel和用于文件操作的FileChannel
  - 多路复用器 Selector
    - 多路复用器提供选择已经就绪的任务的能务，Selector会不断的轮询注册在其上的Channel，如果某个Channel
    上面有新的TCP连接、读和写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以
    获取就绪Channel的集合，进行后续的I/O操作.
    - 一个多路复用器Selector可以同时轮询多个Channel，由于JDK使用了epoll()代替传统的select实现，所以它并没最大
    连接句柄1024/2048的限制。这也就意味着只需要一个线程负责Selector的轮询，就可以接入成成千上万的客户端。
  - 多路复用模型
    - select/poll
      - 将channel注册到selector上，通过轮询channel是否就绪，将就绪的channel返回
    - epoll
      - 将channel注册到selector上，基于回调的方式(类似于监听者模式),告知selector哪些channel已经就绪，然后将就绪的channel返回
    - select/poll和epoll性能分析
      - 对比select/poll和epoll我们发现epoll效率更高
      - 如果select/poll中注册大量的channel，就要不停的轮询每个channel,来判断哪些channel已经就绪，而epoll则不需要轮询
      - jdk1.4是使用select/poll模型,jdk1.5以后把select/poll改为epoll模型