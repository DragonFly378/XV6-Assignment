# Task 1 --> Syscall Tracing

## syscall.c

```diff 
--git a/syscall.c b/syscall.c
index 9105b52..09441db 100644
--- a/syscall.c
+++ b/syscall.c
@@ -106,6 +106,10 @@ extern int sys_uptime(void);
 #ifdef PDX_XV6
 extern int sys_halt(void);
 #endif // PDX_XV6
+#ifdef CS333_P1
:...skipping...
diff --git a/syscall.c b/syscall.c
index 9105b52..09441db 100644
--- a/syscall.c
+++ b/syscall.c
@@ -106,6 +106,10 @@ extern int sys_uptime(void);
 #ifdef PDX_XV6
 extern int sys_halt(void);
 #endif // PDX_XV6
+#ifdef CS333_P1
+// internally, the function prototype must be ’int’ not ’uint’ for sys_date()
+extern int sys_date(void);
+#endif // CS333_P1
 
 static int (*syscalls[])(void) = {
 [SYS_fork]    sys_fork,
@@ -132,6 +136,9 @@ static int (*syscalls[])(void) = {
 #ifdef PDX_XV6
 [SYS_halt]    sys_halt,
 #endif // PDX_XV6
+#ifdef CS333_P1
+[SYS_date]    sys_date,
+#endif // PDX_XV6
 };
 
 #ifdef PRINT_SYSCALLS
@@ -172,6 +179,9 @@ syscall(void)
   num = curproc->tf->eax;
   if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
     curproc->tf->eax = syscalls[num]();
+    #ifdef PRINT_SYSCALLS
:

```


# Task 2 -->  DATE syscall


## Makefile
```diff 
--git a/Makefile b/Makefile
index 6483959..acba3d9 100644
--- a/Makefile
+++ b/Makefile
@@ -1,6 +1,6 @@
 # Set flag to correct CS333 project number: 1, 2, ...
 # 0 == original xv6-pdx distribution functionality
-CS333_PROJECT ?= 0
+CS333_PROJECT ?= 1
 PRINT_SYSCALLS ?= 0
 CS333_CFLAGS ?= -DPDX_XV6
 ifeq ($(CS333_CFLAGS), -DPDX_XV6)
@@ -13,7 +13,7 @@ endif
 
 ifeq ($(CS333_PROJECT), 1)
 CS333_CFLAGS += -DCS333_P1
:
```

## date.c
```diff 
--git a/date.c b/date.c
index cff33a2..8c8a8f8 100644
--- a/date.c
+++ b/date.c
@@ -38,9 +38,9 @@ main(int argc, char *argv[])
   r.hour %= 12;
   if (r.hour == 0) r.hour = 12;
 
-  printf(1, "%s %s%d %s %d %s%d:%s%d:%s%d %s UTC\n", days[day], PAD(r.day), r.day,
-      months[r.month], r.year, PAD(r.hour), r.hour, PAD(r.minute), r.minute,
-      PAD(r.second), r.second, s);
+  printf(1, "%s %s  %d %s%d:%s%d:%s%d %s UTC %d\n", days[day], months[r.month],
+      r.day, PAD(r.hour), r.hour, PAD(r.minute), r.minute, 
+      PAD(r.second), r.second, s, r.year);
 
   exit();
:
```

## syscall.c

```diff 
--git a/syscall.c b/syscall.c
index 9105b52..09441db 100644
--- a/syscall.c
+++ b/syscall.c
@@ -106,6 +106,10 @@ extern int sys_uptime(void);
 #ifdef PDX_XV6
 extern int sys_halt(void);
 #endif // PDX_XV6
+#ifdef CS333_P1
+// internally, the function prototype must be ’int’ not ’uint’ for sys_date()
+extern int sys_date(void);
+#endif // CS333_P1
 
 static int (*syscalls[])(void) = {
 [SYS_fork]    sys_fork,
@@ -132,6 +136,9 @@ static int (*syscalls[])(void) = {
:
```

## syscall.h
```diff 
--git a/syscall.h b/syscall.h
index 7fc8ce1..88f26e9 100644
--- a/syscall.h
+++ b/syscall.h
@@ -22,3 +22,4 @@
 #define SYS_close   SYS_mkdir+1
 #define SYS_halt    SYS_close+1
 // student system calls begin here. Follow the existing pattern.
+#define SYS_date    SYS_halt+1
\ No newline at end of file
```

## sysproc.c

```diff 
--git a/sysproc.c b/sysproc.c
index 98563ea..bea2b63 100644
--- a/sysproc.c
+++ b/sysproc.c
@@ -97,3 +97,17 @@ sys_halt(void)
   return 0;
 }
 #endif // PDX_XV6
+
+#ifdef CS333_P1
+int
+sys_date(void)
+{
+  struct rtcdate *d;
+
+  if(argptr(0, (void*)&d, sizeof(struct rtcdate)) < 0)
:
```

## user.h
```diff 
--git a/user.h b/user.h
index 31d9134..503028d 100644
--- a/user.h
+++ b/user.h
@@ -25,6 +25,9 @@ char* sbrk(int);
 int sleep(int);
 int uptime(void);
 int halt(void);
+#ifdef CS333_P1
+int date(struct rtcdate*);
+#endif // CS333_P1
 
 // ulib.c
 int stat(char*, struct stat*);
 ```
 
 
## usys.S
```diff 
--git a/usys.S b/usys.S
index 0d4eaed..0fbebef 100644
--- a/usys.S
+++ b/usys.S
@@ -30,3 +30,5 @@ SYSCALL(sbrk)
 SYSCALL(sleep)
 SYSCALL(uptime)
 SYSCALL(halt)
+SYSCALL(date)
+
```

# Task 3 --> Ctrl-P
## proc.c
```diff 
--git a/proc.c b/proc.c
index d030537..e513480 100644
--- a/proc.c
+++ b/proc.c
@@ -149,7 +149,9 @@ allocproc(void)
   memset(p->context, 0, sizeof *p->context);
   p->context->eip = (uint)forkret;
 
+  p->start_ticks = ticks;
   return p;
+
 }
 
 //PAGEBREAK: 32
@@ -563,7 +565,14 @@ procdumpP2P3P4(struct proc *p, char *state_string)
 void
:
```

## proc.h
```diff 
--git a/proc.h b/proc.h
index 0a0b4c5..c7ee129 100644
--- a/proc.h
+++ b/proc.h
@@ -49,6 +49,7 @@ struct proc {
   struct file *ofile[NOFILE];  // Open files
   struct inode *cwd;           // Current directory
   char name[16];               // Process name (debugging)
+  uint start_ticks;
 };
 
 // Process memory is laid out contiguously, low addresses first:
 ```
