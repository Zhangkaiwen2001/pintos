# pintos

## 一、前提背景

本篇文章只是我自己一点点对实验的拙见，具体详细还应该看同学大佬的详细介绍

本文参考:

[大佬轩宝博客](https://www.dengzexuan.top/)

[初始代码（未通过实验版本）](https://github.com/ThierrySans/CSCC69-Pintos)

[通过案例的参考代码](https://github.com/NicoleMayer/pintos_project2)

[自己的代码（其实都是根据大佬们的完整代码en嫖的）](https://github.com/Zhangkaiwen2001/pintos)

下面开始我们的实验吧



## 二、环境配置

因为我看到有很多同伴都不知道环境应该如何配置

下面提供一个不用自己配置直接无脑使用的方法——docker

docker可以使用别人搭建好的环境，然后我们直接使用即可

[docker的下载安装可见此链接](https://yeasy.gitbook.io/docker_practice/install/)

安装好后可以直接在pintos文件夹内打开终端，输入命令

```
docker run --rm --name pintos -it -v "$(pwd):/pintos" thierrysans/pintos bash
```

即可进入由thierrysans教授提供的ubuntu环境，然后就可以愉快的跑代码啦

![成功进入ubuntu环境的界面](./1.png)

图为成功进入ubuntu环境的界面

## 三、代码实现

完成了环境配置我们就可以愉快的写代码啦

代码完成主要有下面几个部分

1. [参数传递](###参数传递)

2. 系统调用

   2.1 进程控制系统调用

   2.2 文件操作系统调用

### 1.参数传递

#### 1.1数据结构和方法

In this task, we mainly modify `process.c` and deal with strings. However, to test the correctness of our algorithm in this task, we must also implement `syscall_write` in `syscall.c`.

在这个模块中，我们主要要修改`process.c`文件。为了验证我们的更改是否正确，我们还需要在`syscall.c`文件中加入`syscall_write`函数

##### 1.1.1数据结构

没有用啥复杂的结构体，只是用到了`string`和`int`

#### 1.1.2方法

- `<process.c>`:

  - 更改`process_execute (const char *file_name)`

    - ```c
      process_execute (const char *file_name) 
      {
        char *fn_copy0, *fn_copy1;
        tid_t tid;
      
      
          /* Make a copy of FILE_NAME.
             Otherwise strtok_r will modify the const char *file_name. */
          fn_copy0 = palloc_get_page(0);//palloc_get_page(0)动态分配了一个内存页
          if (fn_copy0 == NULL)//分配失败
              return TID_ERROR;
      
        /* Make a copy of FILE_NAME.
           Otherwise there's a race between the caller and load(). */
        fn_copy1 = palloc_get_page (0);
        if (fn_copy1 == NULL)
        {
          palloc_free_page(fn_copy0);
          return TID_ERROR;
        }
        //把file_name 复制2份，PGSIZE为页大小
        strlcpy (fn_copy0, file_name, PGSIZE);
        strlcpy (fn_copy1, file_name, PGSIZE);
      
      
        /* Create a new thread to execute FILE_NAME. */
        char *save_ptr;
        char *cmd = strtok_r(fn_copy0, " ", &save_ptr);
        
        tid = thread_create(cmd, PRI_DEFAULT, start_process, fn_copy1);
        palloc_free_page(fn_copy0);
        if (tid == TID_ERROR)
        {
          palloc_free_page (fn_copy1); 
          return tid;
        }
          //后续exec系统调用要求,懒得删了...
          /* Sema down the parent process, waiting for child */
        sema_down(&thread_current()->sema);
        if (!thread_current()->success) return TID_ERROR;//can't create new process thread,return error
      
        return tid;
      }
      
      ```

  - 更改`start_process (void *file_name_)`

    - ```c
      start_process (void *file_name_)
      {
        char *file_name = file_name_;
        struct intr_frame if_;
        bool success;
      
      char *fn_copy=malloc(strlen(file_name)+1);
      strlcpy(fn_copy,file_name,strlen(file_name)+1);
      
        /* Initialize interrupt frame */
        memset (&if_, 0, sizeof if_);
        if_.gs = if_.fs = if_.es = if_.ds = if_.ss = SEL_UDSEG;
        if_.cs = SEL_UCSEG;
        if_.eflags = FLAG_IF | FLAG_MBS;
      
        /*load executable. */
        //此处发生改变，需要传入文件名
        char *token, *save_ptr;
        file_name = strtok_r (file_name, " ", &save_ptr);
        success = load (file_name, &if_.eip, &if_.esp);
        
        if (success)
          {
      
          /* Our implementation for Task 1:
            Calculate the number of parameters and the specification of parameters */
          int argc = 0;
          /* The number of parameters can't be more than 50 in the test case */
          int argv[50];
          for (token = strtok_r (fn_copy, " ", &save_ptr); token != NULL; token = strtok_r (NULL, " ", &save_ptr)){
            if_.esp -= (strlen(token)+1);//栈指针向下移动，留出token+'\0'的大小
            memcpy (if_.esp, token, strlen(token)+1);//token+'\0'复制进去
            argv[argc++] = (int) if_.esp;//存储 参数的地址
          }
          push_argument (&if_.esp, argc, argv);//将参数的地址压入栈
           /* Record the exec_status of the parent thread's success and sema up parent's semaphore */
          thread_current ()->parent->success = true;
          sema_up (&thread_current ()->parent->sema);
          }
        /* Free file_name whether successed or failed. */
        palloc_free_page (file_name);
        free(fn_copy);
        if (!success) 
        {
          thread_current ()->parent->success = false;
          sema_up (&thread_current ()->parent->sema);
          thread_exit ();
        }
          
      
        /* Start the user process by simulating a return from an
           interrupt, implemented by intr_exit (in
           threads/intr-stubs.S).  Because intr_exit takes all of its
           arguments on the stack in the form of a `struct intr_frame',
           we just point the stack pointer (%esp) to our stack frame
           and jump to it. */
        asm volatile ("movl %0, %%esp; jmp intr_exit" : : "g" (&if_) : "memory");
        NOT_REACHED ();
      }
      ```

  - 增加`push_argument(void **esp,int argc, int argv[])`

    - ```c
      /* Our implementation for Task 1:
        Push argument into stack, this method is used in Task 1 Argument Pushing */
      void
      push_argument (void **esp, int argc, int argv[]){
        *esp = (int)*esp & 0xfffffffc;
        *esp -= 4;//四位对齐（word-align）下压uint8_t大小
        *(int *) *esp = 0;
          /*下面这个for循环的意义是：按照argc的大小，循环压入argv数组，这也符合argc和argv之间的关系*/
        for (int i = argc - 1; i >= 0; i--)
        {
          *esp -= 4;
          *(int *) *esp = argv[i];
        }
        *esp -= 4;
        *(int *) *esp = (int) *esp + 4;//压入argv[0]的地址
        *esp -= 4;
        *(int *) *esp = argc;
        *esp -= 4;
        *(int *) *esp = 0;
      }
      ```

  - 更改`process_wait (tid_t child_tid)`

    - ```c
      int
      process_wait (tid_t child_tid UNUSED)
      {
        /* Find the child's ID that the current thread waits for and sema down the child's semaphore */
        struct list *l = &thread_current()->childs;
        struct list_elem *temp;
        temp = list_begin (l);
        struct child *temp2 = NULL;
        while (temp != list_end (l))
        {
          temp2 = list_entry (temp, struct child, child_elem);
          if (temp2->tid == child_tid)
          {
            if (!temp2->isrun)
            {
              temp2->isrun = true;
              sema_down (&temp2->sema);
              break;
            } 
            else 
            {
              return -1;
            }
          }
          temp = list_next (temp);
        }
        if (temp == list_end (l)) {
          return -1;
        }
        list_remove (temp);
        return temp2->store_exit;
      }
      ```

      



