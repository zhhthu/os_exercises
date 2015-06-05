# 死锁与IPC(lec 20) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 死锁概念 
 - 尝试举一个生活中死锁实例。
  - 汽车过桥，桥上只有单行道，当双向的汽车同时驶上桥且面对面不肯退让时，双方即陷入死锁。
 - 可重用资源和消耗资源有什么区别？
  - 可重用资源在使用之后可以重新使用；消耗资源在使用之后即不可再次使用。

### 可重用和不可撤销；
 - 资源分配图中的顶点和有向边代表什么含义？
  - 顶点表示进程，有向边表示进程申请资源（进程指向资源时）或者进程占用资源（资源指向进程时）。
 - 出现死锁的4个必要条件是什么？
  - 互斥、持有并等待、非抢占、循环等待

### 死锁处理方法 
 - 死锁处理的方法有哪几种？它们的区别在什么地方？
 - 安全序列的定义是什么？

### 进程的最大资源需要量小于可用资源与前面进程占用资源的总合；
 - 安全、不安全和死锁有什么区别和联系？

### 银行家算法 
 - 什么是银行家算法？
 - 安全状态判断和安全序列是一回事吗？

### 死锁检测 
 - 死锁检测与安全状态判断有什么区别和联系？

> 死锁检测、安全状态判断和安全序列判断的本质就是资源分配图中的循环等待判断。

### 进程通信概念 
 - 直接通信和间接通信的区别是什么？
  本质上来说，间接通信可以理解为两个直接通信，间接通信中假定有一个永远有效的直接通信方。
 - 同步和异步通信有什么区别？
### 信号和管道 
 - 尝试视频中的信号通信例子。
 - 写一个检查本机网络服务工作状态并自动重启相关服务的程序。
 - 什么是管道？

### 消息队列和共享内存 
 - 写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。
 
## 小组思考题

 - （spoc） 每人用python实现[银行家算法](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/bankers-homework.py)。大致输出可参考[参考输出](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/example-output.txt)。除了`YOUR CODE`部分需要填写代码外，在算法的具体实现上，效率也不高，可进一步改进执行效率。

完成后的代码如下：

```
```
import os
import random
import numpy as np
import itertools

class Bankers(object):
    def __init__(self, totalResource):
        #initiating
        self.RESOURCE = totalResource

    def SignProcesses(self, max_, allocated_):
        self.max = max_
        self.allocated = allocated_
        self.need = self.CalcNeed()
        self.avaliable = self.CalcAvaliable()
        self.finished = [False]*len(self.allocated)

    def Difference(self,a,b):
        #return matrix subtracted from a by b
        res = []
        for i in range(len(a)):
            tmp = []
            for j in range(len(a[i])):
                tmp.append(a[i][j]-b[i][j])
            res.append(tmp)
        return res

    def CalcNeed(self):
        #calc request by subtracting signed matrix from max matrix
        return self.Difference(self.max,self.allocated)

    def CalcAvaliable(self):
        """Calc Avaliable Resource"""
        a = self.allocated
        res = []
        for j in range(len(a[0])):
            tmp = 0
            for i in range(len(a)):
                tmp += a[i][j]
            res.append(self.RESOURCE[j] - tmp)
        return res

    def ExecuteProcess(self,index):
        #print 'Dispatching Done...'
        #check if less avaliable than Request
        # YOUR CODE, YOUR ID 2012011272
        #print index
        req = self.need[index];
        for i in range(0,len(req)):
            if req[i]-self.avaliable[i]>0:
                #print 'HOLD: not enough Resource'
                #print"?"
                return False
        #check END here


        #allocating what they need.
        # YOUR CODE, YOUR ID 2012011272
        for i in range(0,len(req)):
            self.allocated[index][i] = req[i]
            self.avaliable[i]=self.avaliable[i]-req[i]
            self.need[index][i]=0
        #allocating END here
        
        #self.print_matrixes();
        return True
    def xiaoyu(self,x,y):
        
        for i in range(0,len(x)):
            if x[i]>y[i]:
                return False
        return True
    def TempSafeCheckAfterRelease(self):
        #check if at least one request can be done after previous process done. not check whole sequances.
        #if every element of Requests can't accepted after previous process done, this mean it is not safe state
        # YOUR CODE, YOU ID
        
       
        fin = {}
        work={}
        n=len(self.need)
        m=len(self.avaliable)
        for i in range(0,m):
            work[i]=self.avaliable[i]
        for i in range(0,n):
            fin[i]=False
        while (1):
            p=False
            for i in range(0,n):
                if (fin[i]==False and self.xiaoyu(self.need[i],work)):
                    p=True
                    for j in range(0,m):
                        work[j]=work[j]+self.need[i][j]
                        #print i,j
                        #self.need[i][j] = 0
                        fin[i]=True
            if (not p):
                for i in range(0,n):
                    if fin[i]==False:
                        return False
                return True

        #check END here
        
        

    def print_matrixes(self):
        print "_____________________________________________"
        print "MAX\t\tAllocated\tNeed"
        for idx in range(len(self.max)):
            print "%s\t%s\t%s" % (self.max[idx],self.allocated[idx], self.need[idx])
        print "_____________________________________________"
        print "Resources:"
        print "Total: %s\tAvailable: %s\n" % (self.RESOURCE, self.avaliable)

    def ReleasingProcess(self,index):
        for i in range(0,len(self.RESOURCE)):
            self.finished[index] = True
            self.allocated[index][i] = 0
        self.avaliable = self.CalcAvaliable()

    def Execute(self):
        i = 0
        # get all permutation of processes
        perm = itertools.permutations(range(procnum), procnum)
        permArray = np.asarray(list(perm))

        for arr in permArray:
            for i in arr:
                if self.finished[i] == False:
                    print "Executing..."
                    print "Request: "
                    print self.need[i]
                    #check if less avaliable than Request
                    if self.ExecuteProcess(i):
                        print "Dispatching Done..."

                        self.print_matrixes()

                        print "-----Releasing Process------"

                        self.ReleasingProcess(i)

                        self.print_matrixes()

                        #check if at least one request can be done after previous process done. not check whole sequances.
                        #if every element of Requests can't accepted after previous process done, this mean it is not safe state
                        if not (self.TempSafeCheckAfterRelease()):
                            print "SAFE STATE: NOT SAFE - There are no sequances can avoid Deadlock"
                            return False
                        processes.append(i)
                    else:
                        print "HOLD: not enough Resource"

                if i == len(self.allocated)-1:
                    i = 0
                else:
                    i += 1

                check = True
                for k in range(0,len(self.allocated)):
                    if self.finished[k] == False:
                        check = False
                        break
                if check == True:
                    return True
                    break
        #every permutation of processes is false
        return False

def getmax():
    res = []
    for j in range(procnum):
        tmp = []
        for i in range(len(total_resources)):
            randnum=random.random()
            remain_max=0
            if j >0:
                remain_max=total_resources[i]
                for k in range(j):
                    remain_max=remain_max-res[k][i]
                if remain_max < 0:
                    remain_max=0
            else:
                remain_max=total_resources[i]
            tmp.append((int)(randnum*remain_max*0.8))
        res.append(tmp)
    return res

def getallocated():
    res = []
    for j in range(procnum):
        tmp = []
        for i in range(len(total_resources)):
            randnum=random.random()
            remain=0
            if j >0:
                remain=max[j][i]
                for k in range(j):
                    remain=remain-res[k][i]
                if remain < 0:
                    remain=0
            else:
                remain=max[j][i]
            tmp.append((int)(randnum*remain))
        res.append(tmp)
    return res

print "start here"
# random seed
seed = 2
random.seed(seed)
# the number of process list
procnum = 3
# the number of type of resource
resnum =  4
# the max total value of resource
restotalval = 30
# the total resources list
total_resources=[]
# the total processes
processes=[]
# set the real total value of resource in total_resources
for i in range(resnum):
    total_resources.append((int)(restotalval*random.random()))
# init the Banker
b = Bankers(total_resources)
# get the max request values of resources from process
max=getmax()
# get the already gotted values of resources from process
allocated=getallocated()
# init need matrix, available vector
b.SignProcesses(max, allocated)
# print all theses matrixes
b.print_matrixes()
# executing Banker algorithm
result=b.Execute()
# show results
if result:
    print "SUCCESS proc lists ",processes
else:
    print "Failed"

# total_resources = [6, 5, 7, 6]
# processes=[]
# b = Bankers(total_resources)
#
# max = [
#     [3, 3, 2, 2],
#     [1, 2, 3, 4],
#     [1, 3, 5, 0],
# ]
# allocated = [
#     [1, 2, 2, 1],
#     [1, 0, 3, 3],
#     [1, 2, 1, 0],
# ]
#
# b.SignProcesses(max, allocated)
# b.print_matrixes()
# result=b.Execute()
# if result:
#     print "SUCCESS proc lists ",processes
# else:
#     print "Failed"
#
#
# total_resources = [10, 10, 8, 5]
# processes=[]
# b = Bankers(total_resources)
# max = [
#         [10, 8, 2,5],
#         [6, 1, 3,1],
#         [3, 1, 4,2],
#         [5, 4, 2,1]
#     ]
# allocated = [
#         [3, 0, 0,3],
#         [1, 1, 2,0],
#         [2, 1, 2,1],
#         [0, 0, 2,0]
#     ]
# b.SignProcesses(max, allocated)
# b.print_matrixes()
# result=b.Execute()
# if result:
#     print "SUCCESS proc lists ",processes
# else:
#     print "Failed"

```
```

 - (spoc) 以小组为单位，请思考在lab1~lab5的基础上，是否能够实现IPC机制，请写出如何实现信号，管道或共享内存（三选一）的设计方案。
 - 队友生病休学了...在ucore的基础上实现信号量的P()、V()操作。
 
 - (spoc) 扩展：用C语言实现某daemon程序，可检测某网络服务失效或崩溃，并用信号量机制通知重启网络服务。[信号机制的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/signal-ex1.c)

 - (spoc) 扩展：用C语言写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。[管道的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/pipe-ex2.c)
