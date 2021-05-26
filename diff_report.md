# Kode yang diubah/ditambah

## user.h

#ifdef CS333_P2
uint getuid(void);
uint getgid(void);
uint getppid(void);

int setuid(uint);
int setgid(uint);
#endif // CS333_P2


## proc.h
  uint uid;
  uint gid;
  
  

## syscall.c

#ifdef CS333_P2
// internally, the
extern int sys_getuid(void);
extern int sys_getgid(void);
extern int sys_getppid(void);
extern int sys_setuid(void);
extern int sys_setgid(void);
#endif // CS333_P2****



## syscall.h

#define SYS_date    SYS_halt+1
#define SYS_getuid  SYS_date+1
#define SYS_getgid    SYS_getuid+1
#define SYS_getppid   SYS_getgid+1
#define SYS_setuid    SYS_getppid+1
#define SYS_setgid    SYS_setuid+1

## proc.c

#ifdef CS333_P2
#include "pdx.h"
#endif //CS333_P@

#ifdef CS333_P2
  p->uid = DEFAULT_UID;
  p->gid = DEFAULT_GID;
  #endif //CS333_P2
  
  np->uid = curproc->uid;
  np->gid = curproc->gid;



## sysproc.c


#ifdef CS333_P2
int 
sys_getuid(void)
{
 return myproc() -> uid;
}
int 
sys_getgid(void)
{
 return myproc() -> gid;
}int 
sys_getppid(void)
{
    if(myproc()->pid==1)
      return myproc()->pid;
    return myproc()->parent->pid;
}
int 
sys_setuid(void)
{
  int capung;
  if(argint(0,&capung) < 0 || capung > 32767 || capung < 0)
    return -1;
  myproc()->uid = (uint)capung;
  return 0;
}
int 
sys_setgid(void)
{
  int capung;
  if(argint(0,&capung) < 0 || capung > 32767 || capung < 0)
    return -1;
  myproc()->gid = (uint)capung;
  return 0;
}

#endif


