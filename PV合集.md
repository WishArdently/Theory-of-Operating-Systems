## PV合集

##### 1、读者－－写者问题。读者-写者问题为数据库访问建立了一个模型。例如,一个系统,其中有许多竞争的进程试图读写其中的数据,多个进程同时读是可以接受的,但如果一个进 程正在更新数据库,则所有的其他进程都不能访问数据库，即使读操作也不行。分别写 出读者优先、写者优先和公平竞争三种情况下的程序。

（1）读者优先

```c++
int count = 0;
semaphore rw = 1;
semaphore rc = 1;
void read(){
	P(rw);
    //write;
    V(rw);
}
void write(){
    P(rc);
    if(count == 0)
        P(rw);
    count ++;
    V(rc);
    //read;
	P(rc);
    count --;
    if(count == 0)
        V(rw);
    V(rc);
}
```

（2）写者优先

```c++
int read_count = 0;
int write_count = 0;
semaphore rw = 1;
semaphore rc = 1;
semaphore wc = 1;
void read(){
    P(rc);
    if(read_count == 0)
        P(rw);
    read_count ++;
    V(rc);
    //read
    P(rc);
    read_count --;
    if(read_count == 0)
        V(rw);
    V(rc);
}
void write(){
	P(wc);
    wc ++;
    if(write_count == 1)
        P(rw);
    V(wc);
    //write
    P(wc);
    wc --;
    if(write_count == 0)
        V(rw);
    V(wc);
}
```

（3）公平竞争

```c++
int count = 0;
semaphore rw = 1;
semaphore lock = 1;
semaphore rc = 1;
void read(){
    while(1){
    	P(lock);
    	P(rc);
    	if(count == 0)
       		P(rw);
   		count ++;
    	V(rc);
    	V(lock);
    	//read
    	P(rc);
    	count --;
    	if(count == 0)
        	V(rw);
    	V(rc);
    }
}
void write(){
    while(1){
		P(lock);
    	P(rw);
    	//write
    	V(rw);
    	V(lock);
    }
}
```

##### 2、有一个理发师，一把理发椅和N把供等候理发的顾客坐的椅子。如果没有顾客，则理发师便在理发师椅子上睡觉；当一个顾客到来时，必须唤醒理发师进行理发；如果理发师正在理发时又有顾客来到，则如果有空椅子可坐，他就坐下来等，如果没有空椅子，他就离开。为理发师和顾客各编一段程序（伪代码）描述他们的行为，要求不能带有竞争条件。

```c++
semaphore mutex = 1;
semaphore comsumer = 0;
semaphore chair = N;
void HairCutter(){
    while(1){
        P(comsumer);
        P(mutex);
        //haircut
        V(mutex);
        V(chair);
    }
}
void Comsumer(){
    while(1){
		P(chair);
        V(consumer);
    }
}
```

##### 3、吸烟者问题。三个吸烟者在一间房间内，还有一个香烟供应者。为了制造并抽掉香烟，每个吸烟者需要三样东西：烟草、纸和火柴。供应者有丰富的货物提供。三个吸烟者中，第一个有自己的烟草，第二个有自己的纸，第三个有自己的火柴。供应者将两样东西放在桌子上，允许一个吸烟者进行对健康不利的吸烟。当吸烟者完成吸烟后唤醒供应者，供应者再放两样东西（随机地）在桌面上，然后唤醒另一个吸烟者。试为吸烟者和供应者编写程序解决问题。

```c++
semaphore finish = 0;
semaphore r1 = 0;	//组合1
semaphore r2 = 0;	//组合2
semaphore r3 = 0;	//组合3
void server(){
    while(1){
    	int i = random(0, 2);
    	if(i == 0){
			V(r1);
    	}
    	else if(i == 1){
			V(r2);
    	}
    	else{
        	V(r3);
    	}
        P(finish);
    }
}
void smoker1(){
    while(1){
        P(r1);
        //smoke
        V(finish);
    }
}
void smoker2(){
	while(1){
        P(r2);
        //smoke
        V(finish);
    }
}
void smoker3(){
    while(1){
        P(r3);
        //smoke
        V(finish);
    }
}
```

##### 4、面包师问题。面包师有很多面包和蛋糕，由n个销售人员销售。每个顾客进店后先取一个号，并且等着叫号。当一个销售人员空闲下来，就叫下一个号。请分别编写销售人员和顾客进程的程序。

```c++
semaphore offer = n;
semaphore order = 0;
void server(){
    while(1){
		P(offer);
        P(order);
        V(offer);
    }
}
void comsumer(){
	while(1){
       V(order);
    }
}
```

##### 5、桌子上有一只盘子，最多可容纳两个水果，每次只能放入或取出一个水果。爸爸专向盘子放苹果（apple），妈妈专向盘子中放桔子（orange）；儿子专等吃盘子中的桔子，女儿专等吃盘子中的苹果。请用P、V操作来实现爸爸、妈妈、儿子、女儿之间的 同步与互斥关系。

```c++
semaphore plate = 2;
//semaphore mutex = 1;
semaphore apple = 0;
semaphore orange = 0;
void father(){
    while(1){
        P(plate);
        //P(mutex);
        //放苹果
        //V(mutex);
        V(apple);
    }
}
void mother(){
	while(1){
        P(plate);
        //P(mutex);
        //放桔子
        //V(mutex);
        V(orange);
    }
}
void son(){
    while(1){
		P(orange);
        //P(mutex);
        //拿走桔子
        //V(mutex);
        V(plate);
    }
}
void dauther(){
	while(1){
		P(apple);
        //P(mutex);
        //拿走苹果
        //V(mutex);
        V(plate);
    }
}
```

##### 6、有一个仓库，可以存放A和B两种产品，仓库的存储空间足够大，但要求： （1）一次只能存入一种产品（A或B）； 2 （2）-N < (A产品数量-B产品数量) < M。 其中，N和M是正整数。试用“存放A”和“存放B”以及P、V操作描述产品A与产品B的入库过程。

```c++
semaphore a = M - 1;
semaphore b = N - 1;
semaphore mutex = 1;
void putA(){
	while(1){
        P(b);
        P(mutex);
        //put A
        V(mutex);
        V(a);
    }
}
void putB(){
	while(1){
        P(a);
        P(mutex);
        //put B
        V(mutex);
        V(b);
    }
}
```

##### 7、三个进程P1、P2、P3 互斥使用一个包含N(N>0)个单元的缓冲区。P1 每次用produce() 生成一个正整数并用put()送入缓冲区某一空单元中;P2 每次用getodd()从该缓冲区中取出一个奇数并用countodd()统计奇数个数;P3 每次用geteven()从该缓冲区中取出一个偶 数并用counteven()统计偶数个数。请用信号量机制实现这三个进程的同步与互斥活动, 并说明所定义信号量的含义。要求用伪代码描述。

```c++
semaphore buf = N;
semaphore mutex = 1;
semaphore odd = 0;
semaphore even = 0;
void P1(){
	while(1){
		int n = produce();
        P(buf);
        P(mutex);
        put(n);
        V(mutex);
        if(n & 1)V(odd);
        else V(even);
    }
}
void P2(){
    while(1){
		P(even);
        P(mutex);
        getodd();
        V(mutex);
        V(buf);
    }
}
void P3(){
    while(1){
        P(odd);
        P(mutex);
        geteven();
        V(mutex);
        V(buf);
    }
}
```

##### 8、在天津大学与南开大学之间有一条弯曲的小路，这条路上每次每个方向上只允许一辆自行车通过。但其中有一个小的安全岛M，同时允许两辆自行车停留，可供两辆自行车已从两端进入小路的情况下错车使用。如图所示。下面的算法可以使来往的自行车均可顺利通过。其中使用了4个信号量，T代表天大路口资源，S代表南开路口资源，L 代表从天大到安全岛一段路的资源，K代表从南开到安全岛一段路的资源。程序如下，请在空白位置处填写适当的PV操作语句，每处空白可能包含若干个PV操作语句。

<img src="C:\Users\16958\AppData\Roaming\Typora\typora-user-images\image-20231226111751079.png" alt="image-20231226111751079" style="zoom:70%;" />

```c++
semaphore T = 1;
semaphore L = 1;
semaphore K = 1;
semaphore S = 1;
void _CollegeTo985(){
	P(T);
    P(L);
    //M
    V(L);
    V(T);
    P(K);
    //到达985
    V(K);
}
void _985ToCollege(){
    P(S);
    P(K);
    //M
    V(K);
	V(S);
    P(L);
    //到达大专
	V(L);
}
```

##### 9、有桥如下图所示，车流如箭头所示，桥上不允许两车交汇，但允许同方向多辆车依次通过（即桥上可以有多个同方向的车）。用P、V操作实现交通管理以防止桥上堵塞。

<img src="C:\Users\16958\AppData\Roaming\Typora\typora-user-images\image-20231226112756908.png" alt="image-20231226112756908" style="zoom:50%;" />

```c++
semaphore bridge = 1;
semaphpre rs = 1;
semaphore rn = 1;
int south = 0;
int north = 0;
void Northward(){
	while(1){
        P(rn);
        if(north == 0)
            P(bridge);
        north ++;
        V(rn);
        //向北
        P(rn);
        north --;
        if(north == 0)
            V(bridge);
        V(rn);
    }
}
void Southward(){
    while(1){
        P(rs);
        if(south == 0)
        	P(bridge);
        south ++;
        V(rs);
        //向南
        P(rs);
        south --;
        if(south == 0)
            V(bridge);
        V(rs);
    }
}
```

##### 10、用 P、V 操作和信号量实现下图中的前趋关系。其中 S1~S5 是 5 个具有同步关系的进 程。

<img src="C:\Users\16958\AppData\Roaming\Typora\typora-user-images\image-20231226114044273.png" alt="image-20231226114044273" style="zoom:70%;" />

```c++
semaphore r1_2 = 0;
semaphore r1_35 = 0;
semaphore r2_4 = 0;
semaphore r3_5 = 0;
semaphore r4_5 = 0;
void S1(){
    while(1){
        //S1
        V(r1_2);
        V(r1_35);
        V(r1_35);
    }
}
void S2(){
	while(1){
        P(r1_2);
        //S2
        V(r2_4);
    }
}
void S3(){
    while(1){
        P(r1_35);
        //S3
        V(r3_5);
    }
}
void S4(){
    while(1){
        P(r2_4);
        //S4
        V(r4_5);
    }
}
void S5(){
    while(1){
        P(r1_35);
        P(r3_5);
        P(r4_5);
        //S5
    }
}
```

##### 11、设有两个优先级相同的进程 P1 和 P2，共享 x、y、z 三个变量，执行代码见下表。信号量 s1 和 s2 的初值均为 0。试问 P1、P2 并发执行后，x、y、z 的值各是多少？给出解题过程。

```c++
semaphore s1 = 0;
semaphore s2 = 0;
int x, y, z;
void P1(){
    y = 1;			//(1)
    y = y + 2;		//(2) y = 3
    V(s1);			
    z = y + 1;		//(3) z = 4
    P(s2);
    y = z + y;		//(7) y = 7 或 (8)(x, y, z) = (7, 14, 11)
}
void P2(){
    x = 1;			//(4)
    x = x + 2;		//(5) x = 3
    P(s1);			
    x = x + y;		//(6) x = 7
    V(s2);			
    z = x + z;		//(8)(x, y, z) = (7, 7, 11) 或 (7)z = 11
}
//(7, 14, 11)或者(7, 7, 11)
```

##### 12、进程 A、B、C、D 为一组合作进程，其前趋图如下图所示，请在下面的程序代码片断中， 对信号量赋初值，并增加 P、V 操作完成进程间同步。

<img src="C:\Users\16958\AppData\Roaming\Typora\typora-user-images\image-20231226115337177.png" alt="image-20231226115337177" style="zoom:67%;" />

```c++
semaphore s1 = 0;
semaphore s2 = 0;
semaphore s3 = 0;
semaphore s4 = 0;
void A(){
    //A
    V(s1);
    V(s1);
}
void B(){
	P(s1);
    //B
    V(s2);
    V(s3);
}
void C(){
	P(s1);
    P(s2);
    //C
    V(s4);
}
void D(){
	P(s3);
    P(s4);
    //D
}
```

一般情况下信号量数目等于边数，但是可根据拓扑关系适当减少信号量个数。
