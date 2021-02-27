JTAG出错Bad JTAG communication问题的解决

下载时候提示：

    ***JLink Error: Bad JTAG communication: Write to IR: Expected 0x1, got 0xF (TAP Command : 10) @ Off 0x5.

调试时候点击全速运行，会提示：


```
***JLink Error: Can not read register 0 (R0) while CPU is running
***JLink Error: Can not read register 1 (R1) while CPU is running
***JLink Error: Can not read register 2 (R2) while CPU is running
***JLink Error: Can not read register 3 (R3) while CPU is running
***JLink Error: Can not read register 4 (R4) while CPU is running
***JLink Error: Can not read register 5 (R5) while CPU is running
***JLink Error: Can not read register 6 (R6) while CPU is running
***JLink Error: Can not read register 7 (R7) while CPU is running
***JLink Error: Can not read register 9 (R15 (PC)) while CPU is running
***JLink Error: Can not read register 8 (CPSR) while CPU is running
***JLink Error: Can not read register 10 (R8_USR) while CPU is running
***JLink Error: Can not read register 11 (R9_USR) while CPU is running
***JLink Error: Can not read register 12 (R10_USR) while CPU is running
***JLink Error: Can not read register 13 (R11_USR) while CPU is running
```


先试着自己写了一个类似的初始化代码，没问题，再去试试别人的代码，也可以调试，没问题，那么说明并不是JTAG硬件上出现问题



因此从HAL库这边着手，通过注释排除法发现执行 HAL_Init();这个函数时候就会出现这个“可以下载程序但是无法调试”的问题

最后判断是HAL_MspInit()函数导致的

```
void HAL_MspInit(void)
{
  /* USER CODE BEGIN MspInit 0 */

  /* USER CODE END MspInit 0 */

  __HAL_RCC_AFIO_CLK_ENABLE();
  __HAL_RCC_PWR_CLK_ENABLE();

  /* System interrupt init*/

  /** NOJTAG: JTAG-DP Disabled and SW-DP Enabled
  */
  __HAL_AFIO_REMAP_SWJ_NOJTAG();

  /* USER CODE BEGIN MspInit 1 */

  /* USER CODE END MspInit 1 */
}
```

进入该函数内，发现在别处重写，通过全工程搜索关键字找到重写的代码

注释语句，问题解决。

    //__HAL_AFIO_REMAP_SWJ_NOJTAG();

CUBEMX 如果说是选择STM32F103，默认生成的代码会自动加上这一句以禁用JTAG