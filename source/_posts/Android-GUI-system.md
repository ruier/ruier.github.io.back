---
title: Android GUI system
date: 2012-04-20 16:21:43
categories: Android
tags: Android
---

## 基于 atmel 的 sam9x5 系列的开发板 Android GUI 系统总结。

目前

*	内核 linux-2.6.39 
*	android 2.3.5

GUI 系统从下至上经过 ：

Linux 内核驱动  ---->  Gralloc HAL  ------>   FrameBufferNativeWindow  ------->  

首先看看内核的驱动主要是 LCD 显示驱动和 overlay 的基于 V4L 的 video 驱动。
9x5 的 LCD 控制器，支持 4 层显示， base/ovr1/HEO/HCC  。分别是 ：

*	base 层，基本的显示层，我们一般的显示都是使用这一层，里面是唯一包含背光控制模块的一层。
*	ovr1 层，overlay 层，数据可以直接往这上面送，它的数据显示在 base 层之上，支持一些 rotation 和色彩变换
*	HEO 层，这一层的功能最多，支持 rotation 以及 scale 等，在驱动中使用 V4L 作成了 video 驱动。
	HEO 和 OVR1 的显示位置可调节，可以设置寄存器支持谁显示在上面
	HCC  层，这是一个硬件光标显示层，在 LCDC 显示在最上面。

目前的 2.6.39 驱动将前面的 3 个都做了出来， HCC 用的较少，所以还没有支持。

### Linux 驱动
下面具体来分析一下这几层的 Linux 驱动程序。

首先在平台设备里面注册了我们需要的两个设备  *base* 和 *ovr1* 的设备。
at91sam9x5_devices.c
```
static struct platform_device at91_lcdc_base_device = {
	.name		= "atmel_hlcdfb_base",
	.id		= 0,
	.dev		= {
				.dma_mask		= &lcdc_dmamask,
				.coherent_dma_mask	= DMA_BIT_MASK(32),
				.platform_data		= &lcdc_data,
	},
	.resource	= lcdc_base_resources,
	.num_resources	= ARRAY_SIZE(lcdc_base_resources),
};
```
```
static struct platform_device at91_lcdc_ovl_device = {
	.name		= "atmel_hlcdfb_ovl",
	.id		= 0,
	.dev		= {
				.dma_mask		= &lcdc_dmamask,
				.coherent_dma_mask	= DMA_BIT_MASK(32),
				.platform_data		= &lcdc_data,
	},
	.resource	= lcdc_ovl1_resources,
	.num_resources	= ARRAY_SIZE(lcdc_ovl1_resources),
};
void __init at91_add_device_lcdc(struct atmel_lcdfb_info *data)
{
	platform_device_register(&at91_lcdc_base_device);
	platform_device_register(&at91_lcdc_ovl_device);
}
```
注册了两个设备 atmel_hlcdfb_base 和 atmel_hlcdfb_ovl 

驱动程序  atmel_hlcdfb.c
将注册的两个设备放在 device tabel 中。他们公用一个设备驱动。
```
static const struct platform_device_id atmelfb_dev_table[] = {
	{ "atmel_hlcdfb_base", (kernel_ulong_t)&dev_data_base },
	{ "atmel_hlcdfb_ovl", (kernel_ulong_t)&dev_data_ovl },
}
```
驱动注册 ：
```
static struct platform_driver atmel_hlcdfb_driver = {
	.remove		= __exit_p(atmel_hlcdfb_remove),
	.suspend	= atmel_hlcdfb_suspend,
	.resume		= atmel_hlcdfb_resume,

	.driver		= {
		.name	= "atmel_hlcdfb",
		.owner	= THIS_MODULE,
	},
	.id_table	= atmelfb_dev_table,
};

static int __init atmel_hlcdfb_init(void)
{
	return platform_driver_probe(&atmel_hlcdfb_driver, atmel_hlcdfb_probe);
}
```

### Android HAL
在 android 中使用了 双缓冲的 机制，所以在内存的分配是 

info->screen_base = dma_alloc_writecombine(info->device, info->fix.smem_len,
					(dma_addr_t *)&info->fix.smem_start, GFP_KERNEL);
fix->smem_start 是 LCD 的真实物理地址，我们使用的是 info->screen_base ，操作的对象是一样的。
DMA 的自循环  ：
sinfo->dma_desc = dma_alloc_writecombine(info->device,
					sinfo->dev_data->dma_desc_size,
					&(sinfo->dma_desc_phys), GFP_KERNEL);
分配 dam_desc 的地址是在 dma_desc_phys 处。update dma 的时候
desc->next = sinfo->dma_desc_phys;
这样将会指向自己。DMA 访问的是物理地址。

