# 文件系统(lec 21) spoc 思考题

## 个人思考题
### 文件系统和文件 
 1. 文件系统的功能是什么？
  - 文件分配、文件管理、数据可靠和安全。

>  负责数据持久保存，功能是数据存储和访问
>  具体功能：文件分配、文件管理、数据可靠和安全

 2. 什么是文件？
  - 文件系统中具有符号名的基本数据单位。

>  文件系统中具有符号名的基本数据单位。

### 文件描述符
 1. 打开文件时，文件系统要维护哪些信息？
  - 文件指针、打开文件计数、访问权限、文件位置和数据缓存

>  文件指针、打开文件计数、访问权限、文件位置和数据缓存

 2. 文件系统的基本数据访问单位是什么？这对文件系统有什么影响？
  - 文件系统的基本数据访问单位是文件，对系统缓存大小等均有影响。

 3. 文件的索引访问有什么特点？如何优化索引访问的速度？

### 目录、文件别名和文件系统种类
 1. 什么是目录？
  - 由文件索引项组成的特殊文件。

>  由文件索引项组成的特殊文件。

 2. 目录的组织结构是什么样的？
  - 树结构、有向图

>  树结构、有向图

 3. 目录操作有哪些种类？
 4. 什么是文件别名？软链接和硬链接有什么区别？
 5. 路径遍历的流程是什么样的？如何优化路径遍历？
 6. 什么是文件挂载？
 7. 为什么会存在大量的文件类型？
  - 因为每个文件系统具备自己的特征，很难有一个文件系统具有所有特征。

### 虚拟文件系统 
 1. 虚拟文件系统的功能是什么？
  - 对上对下的接口、高效访问实现

>  对上对下的接口、高效访问实现

 1. 文件卷控制块、文件控制块和目录项的相互关系是什么？
  - 文件卷控制块每个文件系统有一个，记录文件系统的详细信息。文件控制块和目录项均对应一个文件。
  
 1. 可以把文件控制块放到目录项中吗？这样做有什么优缺点？
  - 可以。这样做的话当文本控制块内容较小时可以节约系统空间。但是当文件控制块内容较大时，查询目录项时会因为目录项太大而增大时间耗费，这时候将文件控制块与目录项分隔开来会更快。


### 文件缓存和打开文件
 1. 文件缓存和页缓存有什么区别和联系？
 1. 为什么要同时维护进程的打开文件表和操作系统的打开文件表？这两个打开文件表有什么区别和联系？为什么没有线程的打开文件表？
  - 因为操作系统有多个进程，每个进程的信息也可能不同，所以在维护进程打开文件表的同时也要维护操作系统的打开文件表。
  - 进程统一管理资源，线程管理代码（指针信息），可以共享资源信息，所以没有必要打开文件表。
 
### 文件分配
 1. 文件分配的三种方式是如何组织文件数据块的？各有什么特征？
 1. UFS多级索引分配是如何组织文件数据块的位置信息的？

### 空闲空间管理和冗余磁盘阵列RAID
 1. 硬盘空闲空间组织和文件分配有什么异同？
 1. RAID-0、1、4和5分别是如何组织磁盘数据块的？各有什么特征？

## 小组思考题
 1. (spoc)完成Simple File System的功能，支持应用程序的一般文件操作。具体帮助和要求信息请看[sfs-homework](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab8/sfs-homework.md)
 - 代码如下：

```
#! /usr/bin/env python

import random
from optparse import OptionParser

DEBUG = False

def dprint(str):
    if DEBUG:
        print str

printOps      = True
printState    = True
printFinal    = True

class bitmap:
    def __init__(self, size):
        self.size = size
        self.bmap = []
        for num in range(size):
            self.bmap.append(0)

    def alloc(self):
        for num in range(len(self.bmap)):
            if self.bmap[num] == 0:
                self.bmap[num] = 1
                return num
        return -1

    def free(self, num):
        assert(self.bmap[num] == 1)
        self.bmap[num] = 0

    def markAllocated(self, num):
        assert(self.bmap[num] == 0)
        self.bmap[num] = 1

    def dump(self):
        s = ''
        for i in range(len(self.bmap)):
            s += str(self.bmap[i])
        return s

class block:
    def __init__(self, ftype):
        assert(ftype == 'd' or ftype == 'f' or ftype == 'free')
        self.ftype = ftype
        # only for directories, properly a subclass but who cares
        self.dirUsed = 0
        self.maxUsed = 32
        self.dirList = []
        self.data    = ''

    def dump(self):
        if self.ftype == 'free':
            return '[]'
        elif self.ftype == 'd':
            rc = ''
            for d in self.dirList:
                # d is of the form ('name', inum)
                short = '(%s,%s)' % (d[0], d[1])
                if rc == '':
                    rc = short
                else:
                    rc += ' ' + short
            return '['+rc+']'
            # return '%s' % self.dirList
        else:
            return '[%s]' % self.data

    def setType(self, ftype):
        assert(self.ftype == 'free')
        self.ftype = ftype

    def addData(self, data):
        assert(self.ftype == 'f')
        self.data = data

    def getNumEntries(self):
        assert(self.ftype == 'd')
        return self.dirUsed

    def getFreeEntries(self):
        assert(self.ftype == 'd')
        return self.maxUsed - self.dirUsed

    def getEntry(self, num):
        assert(self.ftype == 'd')
        assert(num < self.dirUsed)
        return self.dirList[num]

    def addDirEntry(self, name, inum):
        assert(self.ftype == 'd')
        self.dirList.append((name, inum))
        self.dirUsed += 1
        assert(self.dirUsed <= self.maxUsed)

    def delDirEntry(self, name):
        assert(self.ftype == 'd')
        tname = name.split('/')
        dname = tname[len(tname) - 1]
        for i in range(len(self.dirList)):
            if self.dirList[i][0] == dname:
                self.dirList.pop(i)
                self.dirUsed -= 1
                return
        assert(1 == 0)

    def dirEntryExists(self, name):
        assert(self.ftype == 'd')
        for d in self.dirList:
            if name == d[0]:
                return True
        return False

    def free(self):
        assert(self.ftype != 'free')
        if self.ftype == 'd':
            # check for only dot, dotdot here
            assert(self.dirUsed == 2)
            self.dirUsed = 0
        self.data  = ''
        self.ftype = 'free'

class inode:
    def __init__(self, ftype='free', addr=-1, refCnt=1):
        self.setAll(ftype, addr, refCnt)

    def setAll(self, ftype, addr, refCnt):
        assert(ftype == 'd' or ftype == 'f' or ftype == 'free')
        self.ftype  = ftype
        self.addr   = addr
        self.refCnt = refCnt

    def incRefCnt(self):
        self.refCnt += 1

    def decRefCnt(self):
        self.refCnt -= 1

    def getRefCnt(self):
        return self.refCnt

    def setType(self, ftype):
        assert(ftype == 'd' or ftype == 'f' or ftype == 'free')
        self.ftype = ftype

    def setAddr(self, block):
        self.addr = block

    def getSize(self):
        if self.addr == -1:
            return 0
        else:
            return 1

    def getAddr(self):
        return self.addr

    def getType(self):
        return self.ftype

    def free(self):
        self.ftype = 'free'
        self.addr  = -1
        

class fs:
    def __init__(self, numInodes, numData):
        self.numInodes = numInodes
        self.numData   = numData
        
        self.ibitmap = bitmap(self.numInodes)
        self.inodes  = []
        for i in range(self.numInodes):
            self.inodes.append(inode())

        self.dbitmap = bitmap(self.numData)
        self.data    = []
        for i in range(self.numData):
            self.data.append(block('free'))
    
        # root inode
        self.ROOT = 0

        # create root directory
        self.ibitmap.markAllocated(self.ROOT)
        self.inodes[self.ROOT].setAll('d', 0, 2)
        self.dbitmap.markAllocated(self.ROOT)
        self.data[0].setType('d')
        self.data[0].addDirEntry('.',  self.ROOT)
        self.data[0].addDirEntry('..', self.ROOT)

        # these is just for the fake workload generator
        self.files      = []
        self.dirs       = ['/']
        self.nameToInum = {'/':self.ROOT}

    def dump(self):
        print 'inode bitmap ', self.ibitmap.dump()
        print 'inodes       ',
        for i in range(0,self.numInodes):
            ftype = self.inodes[i].getType()
            if ftype == 'free':
                print '[]',
            else:
                print '[%s a:%s r:%d]' % (ftype, self.inodes[i].getAddr(), self.inodes[i].getRefCnt()),
        print ''
        print 'data bitmap  ', self.dbitmap.dump()
        print 'data         ',
        for i in range(self.numData):
            print self.data[i].dump(),
        print ''

    def makeName(self):
        p = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'j', 'k', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z']
        return p[int(random.random() * len(p))]

    def inodeAlloc(self):
        return self.ibitmap.alloc()

    def inodeFree(self, inum):
        self.ibitmap.free(inum)
        self.inodes[inum].free()

    def dataAlloc(self):
        return self.dbitmap.alloc()

    def dataFree(self, bnum):
        self.dbitmap.free(bnum)
        self.data[bnum].free()
        
    def getParent(self, name):
        tmp = name.split('/')
        if len(tmp) == 2:
            return '/'
        pname = ''
        for i in range(1, len(tmp)-1):
            pname = pname + '/' + tmp[i]
        return pname

    def deleteFile(self, tfile):
        if printOps:
            print 'unlink("%s");' % tfile
        inum = self.nameToInum[tfile]

        addr = self.inodes[inum].getAddr()

        if self.inodes[inum].getRefCnt() == 1:
            if self.data[addr].ftype!='free':
                self.data[addr].free()
            self.inodeFree(inum)
        else:
            self.inodes[inum].decRefCnt()



        pname = self.getParent(tfile)
        pinum = self.nameToInum[pname]
        paddr = self.inodes[pinum].getAddr()
        
        self.data[paddr].delDirEntry(tfile)
        self.inodes[pinum].decRefCnt()

    # YOUR CODE, YOUR ID
        # IF inode.refcnt ==1, THEN free data blocks first, then free inode, ELSE dec indoe.refcnt
        # remove from parent directory: delete from parent inum, delete from parent addr
    # DONE
        # finally, remove from files list
        self.files.remove(tfile)
        return 0

    def createLink(self, target, newfile, parent):
       
        #print '--------------'       
    # YOUR CODE, YOUR ID
        # find info about parent
        # is there room in the parent directory?
        # if the newfile was already in parent dir?
        # now, find inumber of target
        # inc parent ref count
        # now add to directory
    # DONE
        #print target
        tnum = self.nameToInum[target]
        taddr = self.inodes[tnum].getAddr()
        
        pnum = self.nameToInum[parent]
        paddr = self.inodes[pnum].getAddr()
        #print paddr,pnum,tnum,taddr
        if self.data[paddr].dirEntryExists(newfile)==True:
            print 'dirEntryExists'
        if self.data[paddr].getFreeEntries()>0:
            self.data[paddr].addDirEntry(newfile,tnum)
            self.inodes[pnum].incRefCnt()
            self.inodes[tnum].incRefCnt()
        else:
            print 'parent directory < 0'
        #print '--------'
        return tnum

    def createFile(self, parent, newfile, ftype):
        inum = self.nameToInum[parent]
        addr = self.inodes[inum].addr
        if not self.data[addr].dirEntryExists(newfile):
            newinum = self.inodeAlloc()
            
            if ftype=='d':
                newdnum = self.dataAlloc()
                self.data[addr].addDirEntry(newfile,newinum)
                self.inodes[inum].incRefCnt()
                newino = self.inodes[newinum]
                newino.setType('d')
                newino.incRefCnt()
                newino.setAddr(newdnum)
                self.data[newdnum].setType('d')
                self.data[newdnum].addDirEntry('.',newinum)
                self.data[newdnum].addDirEntry('..',inum)
                
            if ftype=='f':
                self.data[addr].addDirEntry(newfile,newinum)
                self.inodes[inum].incRefCnt()
                newino = self.inodes[newinum]
                newino.setType('f')
                newino.setAddr(-1)
                #newino.setType('f')
                #newino.setAddr('-1')

        #print dname
        


    # YOUR CODE, YOUR ID
        # find info about parent
        # is there room in the parent directory?
        # have to make sure file name is unique
        # find free inode
        # if a directory, have to allocate directory block for basic (., ..) info
        # now ok to init inode properly
        # inc parent ref count
        # and add to directory of parent
    # DONE
        return newinum

    def writeFile(self, tfile, data):
        inum = self.nameToInum[tfile]
        curSize = self.inodes[inum].getSize()   # 1 = ,  0 = -1
        dprint('writeFile: inum:%d cursize:%d refcnt:%d' % (inum, curSize, self.inodes[inum].getRefCnt()))
        if curSize:
            return -1
        else:
            dnum = self.dbitmap.alloc()
            if dnum == -1:
                return -1
            self.data[dnum].setType('f')
            self.data[dnum].addData(data)
            self.inodes[inum].addr = dnum
            return 0
    # YOUR CODE, YOUR ID
        # file is full?
        # no data blocks left
        # write file data
    # DONE

        if printOps:
            print 'fd=open("%s", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);' % tfile
        return 0
            
    def doDelete(self):
        dprint('doDelete')
        if len(self.files) == 0:
            return -1
        dfile = self.files[int(random.random() * len(self.files))]
        dprint('try delete(%s)' % dfile)
        return self.deleteFile(dfile)

    def doLink(self):
        dprint('doLink')
        if len(self.files) == 0:
            return -1
        parent = self.dirs[int(random.random() * len(self.dirs))]
        nfile = self.makeName()

        # pick random target
        target = self.files[int(random.random() * len(self.files))]

        # get full name of newfile
        if parent == '/':
            fullName = parent + nfile
        else:
            fullName = parent + '/' + nfile

        dprint('try createLink(%s %s %s)' % (target, nfile, parent))
        inum = self.createLink(target, nfile, parent)
        if inum >= 0:
            self.files.append(fullName)
            self.nameToInum[fullName] = inum
            if printOps:
                print 'link("%s", "%s");' % (target, fullName)
            return 0
        return -1
    
    def doCreate(self, ftype):
        dprint('doCreate')
        parent = self.dirs[int(random.random() * len(self.dirs))]
        nfile = self.makeName()
        if ftype == 'd':
            tlist = self.dirs
        else:
            tlist = self.files

        if parent == '/':
            fullName = parent + nfile
        else:
            fullName = parent + '/' + nfile

        dprint('try createFile(%s %s %s)' % (parent, nfile, ftype))
        inum = self.createFile(parent, nfile, ftype)
        if inum >= 0:
            tlist.append(fullName)
            self.nameToInum[fullName] = inum
            if parent == '/':
                parent = ''
            if ftype == 'd':
                if printOps:
                    print 'mkdir("%s/%s");' % (parent, nfile)
            else:
                if printOps:
                    print 'creat("%s/%s");' % (parent, nfile)
            return 0
        return -1

    def doAppend(self):
        dprint('doAppend')
        if len(self.files) == 0:
            return -1
        afile = self.files[int(random.random() * len(self.files))]
        dprint('try writeFile(%s)' % afile)
        data = chr(ord('a') + int(random.random() * 26))
        rc = self.writeFile(afile, data)
        return rc

    def run(self, numRequests):
        self.percentMkdir  = 0.40
        self.percentWrite  = 0.40
        self.percentDelete = 0.20
        self.numRequests   = numRequests

        print 'Initial state'
        print ''
        self.dump()
        print ''
        
        for i in range(numRequests):
            if printOps == False:
                print 'Which operation took place?'
            rc = -1
            while rc == -1:
                r = random.random()
                if r < 0.3:
                    rc = self.doAppend()
                    dprint('doAppend rc:%d' % rc)
                elif r < 0.5:
                    rc = self.doDelete()
                    dprint('doDelete rc:%d' % rc)
                elif r < 0.7:
                    rc = self.doLink()
                    dprint('doLink rc:%d' % rc)
                else:
                    if random.random() < 0.75:
                        rc = self.doCreate('f')
                        dprint('doCreate(f) rc:%d' % rc)
                    else:
                        rc = self.doCreate('d')
                        dprint('doCreate(d) rc:%d' % rc)
            if printState == True:
                print ''
                self.dump()
                print ''
            else:
                print ''
                print '  State of file system (inode bitmap, inodes, data bitmap, data)?'
                print ''

        if printFinal:
            print ''
            print 'Summary of files, directories::'
            print ''
            print '  Files:      ', self.files
            print '  Directories:', self.dirs
            print ''

#
# main program
#
parser = OptionParser()

parser.add_option('-s', '--seed',        default=0,     help='the random seed',                      action='store', type='int', dest='seed')
parser.add_option('-i', '--numInodes',   default=8,     help='number of inodes in file system',      action='store', type='int', dest='numInodes') 
parser.add_option('-d', '--numData',     default=8,     help='number of data blocks in file system', action='store', type='int', dest='numData') 
parser.add_option('-n', '--numRequests', default=10,    help='number of requests to simulate',       action='store', type='int', dest='numRequests')
parser.add_option('-r', '--reverse',     default=False, help='instead of printing state, print ops', action='store_true',        dest='reverse')
parser.add_option('-p', '--printFinal',  default=False, help='print the final set of files/dirs',    action='store_true',        dest='printFinal')

(options, args) = parser.parse_args()

print 'ARG seed',        options.seed
print 'ARG numInodes',   options.numInodes
print 'ARG numData',     options.numData
print 'ARG numRequests', options.numRequests
print 'ARG reverse',     options.reverse
print 'ARG printFinal',  options.printFinal
print ''

random.seed(options.seed)

if options.reverse:
    printState = False
    printOps   = True
else:
    printState = True
    printOps   = False


printOps   = True
printState = True

printFinal = options.printFinal

#
# have to generate RANDOM requests to the file system
# that are VALID!
#

f = fs(options.numInodes, options.numData)

#
# ops: mkdir rmdir : create delete : append write
#

f.run(options.numRequests)
```

 1. (spoc)FAT、UFS、YAFFS、NTFS这几种文件系统中选一种，分析它的文件卷结构、目录结构、文件分配方式，以及它的变种。
  wikipedia上的文件系统列表参考
  - http://en.wikipedia.org/wiki/List_of_file_systems
  - http://en.wikipedia.org/wiki/File_system
  - http://en.wikipedia.org/wiki/List_of_file_systems

  请同学们依据自己的选择，在下面链接处回复分析结果。
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=416 FAT文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=417 NTFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=418 UFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=419 ZFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=420 YAFFS文件系统分析]
