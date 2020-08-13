---
layout: post
author: 侬工
---
opencv默认的图像数据格式是BGR24，而海康摄像头视频编码格式是H264，视频帧数据格式是YV12，因此需要将YV12转换为BGR24 ，同时也会说明下怎么转换为虹软SDK支持的其它格式。

a.YV12 To BGR24

void yv12ToBGR24(unsigned char* yv12, unsigned  char* bgr24, int width, int height)
{
unsigned char* y_yv12 = yv12;
unsigned char* v_yv12 = yv12 + width*height;
unsigned char* u_yv12 = yv12 + width*height + width*height / 4;

unsigned char* b = bgr24;
unsigned char* g = bgr24 + 1;
unsigned char* r = bgr24 + 2;

int yIndex, uIndex, vIndex;
for (int i = 0; i < height; ++i)
{
    for (int j = 0; j < width; ++j)
    {
        yIndex = i * width + j;
        vIndex = (i / 2) * (width / 2) + (j / 2);
        uIndex = vIndex;

        *b = (unsigned char)(y_yv12[yIndex] + 1.732446 * (u_yv12[vIndex] - 128));
        *g = (unsigned char)(y_yv12[yIndex] - 0.698001 * (u_yv12[uIndex] - 128) - 0.703125 * (v_yv12[vIndex] - 128));
        *r = (unsigned char)(y_yv12[yIndex] + 1.370705 * (v_yv12[uIndex] - 128));

        b += 3;
        g += 3;
        r += 3;
    }
}

b.YV12 To I42

void yv12ToI420(unsigned char yv12, unsigned char i420, int width, int height)
{unsigned char* y_yv12 = yv12;
unsigned char* v_yv12 = yv12 + width*height;
unsigned char* u_yv12 = yv12 + width*height + width*height / 4;

unsigned char* y_i420 = i420;
unsigned char* u_i420 = i420 + width*height;
unsigned char* v_i420 = i420 + width*height + width*height / 4;

memcpy(i420, yv12, width*height);
memcpy(v_i420, v_yv12, width*height / 4);
memcpy(u_i420, u_yv12, width*height / 4);
}


c.YV12 To NV2
void yv12ToNV21(unsigned char yv12, unsigned char nv21, int width, int height)
{unsigned char* y_yv12 = yv12;
unsigned char* v_yv12 = yv12 + width*height;
unsigned char* u_yv12 = yv12 + width*height + width*height / 4;

unsigned char* y_nv21 = nv21;
unsigned char* v_nv21 = nv21 + width*height;
unsigned char* u_nv21 = nv21 + width*height + 1;

memcpy(nv21, yv12, width*height);

for (int i = 0; i < width*height / 4; ++i)
{
    *v_nv21 = *v_yv12;
    *u_nv21 = *u_yv12;

    v_nv21 += 2;
    u_nv21 += 2;

    ++v_yv12;
    ++u_yv12;
}
 }

d.YV12 To NV12
void yv12ToNV12(unsigned char yv12, unsigned char nv12, int width, int height)
{unsigned char* y_yv12 = yv12;
unsigned char* v_yv12 = yv12 + width*height;
unsigned char* u_yv12 = yv12 + width*height + width*height / 4;

unsigned char* y_nv12 = nv12;
unsigned char* u_nv12 = nv12 + width*height;
unsigned char* v_nv12 = nv12 + width*height + 1;

memcpy(nv12, yv12, width*height);

for (int i = 0; i < width*height / 4; ++i)
{
    *v_nv12 = *v_yv12;
    *u_nv12 = *u_yv12;

    v_nv12 += 2;
    u_nv12 += 2;

    ++v_yv12;
    ++u_yv12;
}
  }

e.YV12 To YUYV

void yv12ToYUYV(unsigned char yv12, unsigned char yuyv, int width, int height)
{unsigned char* y_yv12 = yv12;
unsigned char* v_yv12 = yv12 + width*height;
unsigned char* u_yv12 = yv12 + width*height + width*height / 4;

unsigned char* y_yuyv = yuyv;
unsigned char* u_yuyv = yuyv + 1;
unsigned char* v_yuyv = yuyv + 3;

for (int i = 0; i < width; ++i)
{
    for (int j = 0; j < height; ++j)
    {
        *y_yuyv = *y_yv12;

        y_yuyv += 2;
        ++y_yv12;
    }
}

for (int j = 0; j < height / 2; ++j)
{
    for (int i = 0; i < width / 2; ++i)
    {
        *u_yuyv = *u_yv12;
        *(u_yuyv + width * 2) = *u_yv12;
        u_yuyv += 4;
        ++u_yv12;

        *v_yuyv = *v_yv12;
        *(v_yuyv + width * 2) = *v_yv12;
        v_yuyv += 4;
        ++v_yv12;
    }
    u_yuyv += width * 2;
    v_yuyv += width * 2;
}
 }

