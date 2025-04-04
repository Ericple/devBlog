+++
date = '2025-03-13T11:57:18+08:00'
draft = false
title = '从OpenScope到手撕cur光标描述文件'
+++

> 本文将介绍我通过读取cur光标描述文件的二进制数据后，将其通过canvas转换为base64图像再应用到HTML中的cursor属性的过程。
> 对大部分人来说这个操作可能有些奇怪，因为css是可以直接设置cursor: url("xxxx/xxx.cur")来应用cur的，但很遗憾，我这个需求比较特殊，
> 不然也不必如此大费周章了……。不过也好，多熟悉一种格式，也是对自己技术能力的提升。

## 一些可以被跳过的故事

两年前，由于开发需要，我将主力笔记本的系统换到了Ubuntu。当时我在VATSIM模拟飞行联网平台当管制员，所使用的管制软件是来自西班牙的一名程序员的[EuroScope](https://www.euroscope.hu/wp/)。
这款软件很完备，它拥有几乎所有真实管制员所需的功能特性，同时还有强大的插件系统和庞大的插件生态。唯一的缺点是，这个闭源软件只支持Windows操作系统。

我尝试过给开发者发邮件询问他们是否会支持其它平台，得到了否定的答复，当时血气方刚，立志要写一款我们自己的管制软件。于是我开始着手准备自行开发一个管制软件。当时对Electron比较熟悉，并且有一个UniACARS模拟飞行数据监测软件的经验，也是用Electron写的，因此采用了Electron+Vue的方式来写第一版的软件，取名叫[OpenScope](https://github.com/Ericple/openscope-project)，并开源在我的GitHub上。

这个项目得到了小规模的支持，VATSIM区域分部中的许多人给予了我一定的技术帮助，当时在对VATSIM某区域分部的扇区进行大量逆向工作后，实现了部分扇区加载的功能，同时实现了显示vatsim-data.json飞行员数据的功能。但就要开发连接fsd服务器功能的时候，遇到了非技术瓶颈——VATSIM禁止非认证的连飞、管制软件接入VATSIM的fsd连飞服务器。我曾尝试过交涉，也向VATSIM技术部门提交过工单，但最后都杳无音讯，石沉大海。最终这个项目不了了之了。

最近除了维护常规软件以外没有什么事情可做，又意外发现Tauri出了2.0版本，它的跨平台能力甚至比Electron还要好。一开始OpenScope技术选型没选它的原因主要是它还没有出1.0版本，而现在情况发生了变化，于是我就想用Tauri来重新写一写OpenScope，并不断完善它的功能。

## 为什么要手撕cur文件

我在三亚扇中发现了一些EuroScope插件，其中最主要的插件是TopSky.dll，它的存在让扇区在OpenScope和EuroScope的加载中出现了很大的差别，因此我必须考虑对它进行分析逆向，并提供可能所需的API以满足在新环境中重新开发实现其功能的需求。其中一个重要功能，就是TopSky可以修改软件内的光标形式。TopSky插件附带6个cur文件以供使用，但我设计的OpenScope插件系统是插件外置的，而Tauri使用了Vite进行打包，这意味着我无法在css里直接通过绝对路径引用这些cur文件，当然，相对路径也不行。最终，我不得不尝试手撕cur文件，并将它转换为base64图像，进而通过`url()`的方式来实现光标替换。

## cur文件格式解析

> 本文以EuroScope的TopSky插件中提供的`TopSkyCursorCross.cur`为例，通过查看其Binary结构来解析cur文件格式。
> 由于笔者是在Tauri环境下，所以使用tauri提供的方法读取文件。如果你想边看边操作，可以使用其它任意方法读取文件，只要最终可以读取到ArrayBuffer即可。

`.cur`文件是Windows使用的光标文件，它与`.ico`图标采用了一样的结构，但存在一些细微的差异（真的很细微，细微到你可以按完全一致的方式读取这两种文件|笑）。

`.cur`文件可以大致划分为三个区域，分别为：

- 文件头
- 目录
- 图像数据

### 文件头

文件头为6个字节，包含3个数据，每个数据占用2字节。因此在DataView中，我们可以使用`getUint16`方法来读取这些数据。

| 偏移量 | bytes | 描述                                                           |
| ------ | ----- | -------------------------------------------------------------- |
| 0      | 2     | 保留，值固定为00                                               |
| 2      | 2     | 文件类型，`1`表示此文件为`.ico`图标，`2`表示此文件为`.cur`光标 |
| 4      | 2     | 图像数量，此文件包含的图像数量，一般都是1                      |

### 文件目录

文件目录一共占16个字节，包含8个数据，数据占用各不相同

| 偏移量 | bytes | 描述                                                       |
| ------ | ----- | ---------------------------------------------------------- |
| 0      | 1     | 图像宽度（以像素为单位）                                   |
| 1      | 1     | 图像高度（以像素为单位）                                   |
| 2      | 1     | 调色板颜色数（0 表示没有调色板，适用于 24 位或 32 位图像） |
| 3      | 1     | 保留，值固定为00                                           |
| 4      | 2     | 热点 X 坐标（仅适用于 `.cur` 文件，表示光标的点击位置）    |
| 6      | 2     | 热点 Y 坐标（仅适用于 `.cur` 文件）                        |
| 8      | 4     | 图像数据大小（以字节为单位）                               |
| 12     | 4     | 图像数据在文件中的偏移量                                   |

### 图像数据

图像数据没有固定的字节数，但它包含一个固定的信息头，信息头占用40个字节

| 偏移量 | bytes | 描述                 |
| ------ | ----- | -------------------- |
| 0      | 40    | BMP 信息头           |
| 40     | /     | 调色板数据（如果有） |
| /      | /     | 图像像素数据         |

#### BMP信息头

BMP信息头作为图像数据的开头，占用40个字节，其中有11条数据，表示图像的各种信息。其中大部分是4个字节的小端序数据，因此需要使用`getUint32(data, true)`来读取它们。

| 偏移量 | bytes | 描述                         |
| ------ | ----- | ---------------------------- |
| 0      | 4     | 信息头大小（固定`40`）       |
| 4      | 4     | 图像宽度（以像素为单位）     |
| 8      | 4     | 图像高度（以像素为单位）     |
| 12     | 2     | 颜色平面数（固定为`1`）      |
| 14     | 2     | 每像素位数                   |
| 16     | 4     | 压缩方式，为`0`时表示未压缩  |
| 20     | 4     | 图像数据大小（以字节为单位） |
| 24     | 4     | 水平分辨率                   |
| 28     | 4     | 垂直分辨率                   |
| 32     | 4     | 调色板颜色数                 |
| 36     | 4     | 重要颜色数                   |

## cur文件读取

> 由于从场景出发，插件内的基本都是1bit黑白双色图标，因此这里只做双色读取示范，其它调色板的以后有空再补充吧（笑

废话不多，直接上代码吧

```
/**
 * 以二进制方式读取cur文件并解析为Base64编码的图像数据
 */
export async function loadCurFileToBase64(uri: string) {
    const ui8 = await readFile(uri); // 读取为Uint8Array，如果你可以直接读取为ArrayBuffer的话也可以直接读为ArrayBuffer，反正后面也要转换
    const buffer = ui8.buffer; // 因为我读取为Uint8Array，所以这里直接使用buffer转换为ArrayBuffer
    const header = new DataView(buffer.slice(0, 6)); // 通过分割buffer的前6个字节，获取文件头信息
    const dir = new DataView(buffer.slice(6, 22)); // 接下来的16个字节为文件目录信息
    const imageDataOffset = dir.getUint32(12, true); // 获取图像数据的偏移量
    const bmpHeader = new DataView(buffer.slice(imageDataOffset, imageDataOffset + 40)); // 获取位图头信息
    const imageBuffer = new DataView(buffer.slice(imageDataOffset + 40)); // 获取图像数据
    const fileType = header.getUint16(2, true); // 获取文件类型，由于ico和cur是共用一种的，为了确保这里只加载cur文件，可以使用这个类型字节来判断是否需要加载
    const imageContain = header.getUint16(4, true); // 获取图像包含的图像数量，这个不重要，一般情况下都是1，我们也先不管这个，读出来备用
    if (fileType != 2 || imageContain == 0) { // 判断文件类型是否正确，以及是否包含图像数据
        throw new Error(`File is not typeof cur or contains no image data.`);
    }
    const width = dir.getUint8(0); // 获取图像宽度
    const height = dir.getUint8(1); // 获取图像高度
    const palletColorCount = bmpHeader.getUint32(32, true); // 获取调色板颜色数量
    const colorPallet: IColor[] = []; // 初始化调色板颜色数组，用于存储调色板颜色信息
    for (let i = 0; i < palletColorCount; i++) { // 读取调色板颜色，调色板以BGR(A或0)的顺序存储，我也不知道为什么第四位全都是0，如果你的第四位不是0的话，可以把第四位当作alpha读取
        const offset = i * 4;
        const b = imageBuffer.getUint8(offset);
        const g = imageBuffer.getUint8(offset + 1);
        const r = imageBuffer.getUint8(offset + 2);
        colorPallet.push({r, g, b});
    }
    const bytesPerRow = Math.ceil(width / 8); // 一个像素用一个bit表示，每个字节有8个像素，计算每行的字节数
    const paddedBytesPerRow = bytesPerRow + (4 - (bytesPerRow % 4)) % 4; // 由于每行字节必须是4的倍数，当字节数不是4的倍数时，需要填充0，因此需要计算填充后每行的字节数
    const imageDataClamped = new Uint8ClampedArray(4 * width * height);
    const pixelOffset = palletColorCount * 4;// 调色板数据4个字节一组，计算占用的字节数，用于找到像素数据的偏移量
    for (let y = 0; y < height; y++) { // 逐个像素读取
        for (let x = 0; x < width; x++) {
            const byteIndex = Math.floor(x / 8);
            const bitIndex = 7 - (x % 8);
            const byte = imageBuffer.getUint8(pixelOffset + y * paddedBytesPerRow + byteIndex);
            const pixelValue = (byte >> bitIndex) & 1; // 像素值指向调色板中的索引
            const color = colorPallet[pixelValue]; // 根据索引获取颜色
            if (color == undefined) continue; // 如果颜色无效，则跳过当前像素
            const imageDataOffset = ((height - y - 1) * width + x) * 4;
            imageDataClamped[imageDataOffset] = color.r;
            imageDataClamped[imageDataOffset + 1] = color.g;
            imageDataClamped[imageDataOffset + 2] = color.b;
            if (pixelValue == 0) // 这里是额外的代码，用于将黑色像素设置为透明，以满足光标显示需求。如果你没有这个需求，这个if可以不写，直接设置为255即可。
                imageDataClamped[imageDataOffset + 3] = 0;
            else
                imageDataClamped[imageDataOffset + 3] = 255;
        }
    }
    // 下面的代码就是正常调用canvas，将图像数据绘制后转换为base64.
    const imageData = new ImageData(imageDataClamped, width, height);
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    if (ctx == null) throw new Error("Can't load context from canvas");
    canvas.width = width;
    canvas.height = height;
    ctx.putImageData(imageData, 0, 0);
    const d = new CursorB64();
    d.value = canvas.toDataURL("image/png"); // 获取base64数据
    d.centerX = width / 2;
    d.centerY = height / 2;
    return d;
}
```
