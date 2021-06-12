---
title: C语言利用iniparser库解析INI文件
date: 2021-06-12 11:31:42
categories: Coding
tags: [ini,opensource]
top:
---

没什么好说的。  

<!-- more -->

:point_right:[iniparser](https://github.com/ndevilla/iniparser)  
`Linux`应用程序编程的例子：  

```c
#include <stdio.h>
#include "iniparser.h"

#define CONFIG_NAME "RTK_4K2K_VBY1_1Seg8Port_new.ini"

#define INIFILE

int main(void)
{
  dictionary * ini;
  ini = iniparser_load(CONFIG_NAME);

#ifdef INIFILE
  FILE * panel_ini = NULL;
  panel_ini = fopen(CONFIG_NAME,"w");
  if(NULL == panel_ini)
  {
    printf("open %s fail!\n", CONFIG_NAME);
    return -1;
  }
  iniparser_set(ini,"PANEL:BACKLIGHT_PWM_MAX","199");
  iniparser_dump_ini(ini, panel_ini);
  fclose(panel_ini);
#else
  iniparser_dump_ini(ini, stdout);
#endif

  iniparser_freedict(ini);

  return 0;
}
```

`iniparser`支持读写，但是u-boot不能调用C标准库，移植的话比较麻烦，要重新实现这些用到的库函数。  
有一个比较热门的开源库是[inih](https://github.com/benhoyt/inih)，但是只发现读（解析）操作，没找到是怎么写的。  
