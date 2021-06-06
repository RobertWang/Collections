> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/bbjjll0331/article/details/5829050?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_v2~rank_aggregation-17-5829050.pc_agg_rank_aggregation&utm_term=linux%E7%BB%88%E7%AB%AF%E6%98%BE%E7%A4%BA%E5%9B%BE%E7%89%87&spm=1000.2123.3001.4430)

首先需要安装 JPEG 库（安装包：jpegsrc.v6b.tar）

下载地址：[http://download.csdn.net/source/2637188](http://d.download.csdn.net/down/2637188/bbjjll0331)

编译并安装 Jpeg 格式支持文件,

   tar zvxf jpegsrc.v6b.tar.gz

   ./configure --enable-shared --enable-static 

    make

    make install

//===================  
   编译 jpeg 时报错  
　　/usr/bin/install -c -m 644 ./cjpeg.1  
　　/usr/local/man/man1/cjpeg.1  
　　/usr/bin/install: cannot create regular file  
　　`/usr/local/man/man1/cjpeg.1': No such file or directory

　　缺少 / usr/local/man 目录及 man1 子目录，新建后重新编译。  
　　shell> mkdir /usr/local/man  
　　shell> mkdir /usr/local/man/man1  
//=====================

代码显示：

/* * $Id: fv.c * $Desp: draw jpeg to framebuffer * $Author: Tower * $Date: Sat July 21 17:15:49 CST 2010 */ #include <stdio.h> #include <stdlib.h> #include <string.h> #include <fcntl.h> #include <linux/fb.h> #include <sys/types.h> #include <sys/stat.h> #include <sys/mman.h> #include <jpeglib.h> #include <jerror.h> #define FB_DEV "/dev/fb0" struct color { unsigned char r; unsigned char g; unsigned char b; } __attribute__((packed)); struct fb_var_screeninfo var; struct fb_fix_screeninfo fix; struct pixel_t { unsigned char b; unsigned char g; unsigned char r; unsigned char a; }; /***************** function declaration ******************/ void usage(char *msg); int fb_open(char *fb_device); int fb_close(int fd); int fb_stat(int fd, int *width, int *height, int *depth); void *fb_mmap(int fd, unsigned int screensize); int fb_munmap(void *start, size_t length); /************ function implementation ********************/ int main(int argc, char *argv[]) { /* * declaration for jpeg decompression */ struct jpeg_decompress_struct cinfo; struct jpeg_error_mgr jerr; FILE *infile; unsigned char *buffer; int fbdev; int i,j; unsigned int w, h; unsigned short bpp; struct color p; void *data, *save; struct pixel_t **fbp; unsigned int fb_width; unsigned int fb_height; unsigned int fb_depth; unsigned int screensize; if (argc < 2) { fprintf(stderr, "Usage: .../n"); return -1; } /* * Ioctl the framebuff */ fbdev = open(FB_DEV, O_RDWR); if (fbdev == -1) { perror("open"); return -1; } ioctl(fbdev, FBIOGET_FSCREENINFO, &fix); ioctl(fbdev, FBIOGET_VSCREENINFO, &var); fb_stat(fbdev, &fb_width, &fb_height, &fb_depth); screensize = fb_width * fb_height; data = mmap(NULL, fix.smem_len, PROT_READ | PROT_WRITE, MAP_SHARED, fbdev, 0); save = malloc(fix.smem_len); memcpy(save, data, fix.smem_len); fbp = (struct pixel_t **)malloc(sizeof(struct pixel_t *) * var.yres); for (i = 0; i < var.yres; ++i) { fbp[i] = (struct pixel_t *)(save + i * var.xres * var.bits_per_pixel / 8); } /* * open input jpeg file */ if ((infile = fopen(argv[1], "rb")) == NULL) { fprintf(stderr, "open %s failed/n", argv[1]); exit(-1); } /* * init jpeg decompress object error handler */ cinfo.err = jpeg_std_error(&jerr); jpeg_create_decompress(&cinfo); /* * bind jpeg decompress object to infile */ jpeg_stdio_src(&cinfo, infile); /* * read jpeg header */ jpeg_read_header(&cinfo, TRUE); /* * decompress process. * note: after jpeg_start_decompress() is called * the dimension infomation will be known, * so allocate memory buffer for scanline immediately */ jpeg_start_decompress(&cinfo); if ((cinfo.output_width > fb_width) || (cinfo.output_height > fb_height)) { printf("too large JPEG file,cannot display/n"); return (-1); } /* printf("cinfo.output_width=%d/tcinfo.output_height=%d/tcinfo.output_components=%d/n",cinfo.output_width,cinfo.output_height,cinfo.output_components); printf("cinfo.output_scanline=%d/n",cinfo.output_scanline); */ /* * display the JPEG on 32bit/linux */ buffer = (unsigned char *) malloc(cinfo.output_width*cinfo.output_components); memset(buffer,0,cinfo.output_width*cinfo.output_components); w=cinfo.output_width; h=cinfo.output_height; // printf("w=%d,h=%d/n",w,h); for (i = 0; i < h; ++i) { unsigned char *tmpbuf=buffer; jpeg_read_scanlines(&cinfo, &tmpbuf, 1); for (j = 0; j <w; ++j) { memcpy(&p,tmpbuf,sizeof(p)); fbp[i][j].r = p.r; fbp[i][j].g = p.g; fbp[i][j].b = p.b; tmpbuf+=3; } } memcpy(data, save, fix.smem_len); /* * finish decompress, destroy decompress object */ jpeg_finish_decompress(&cinfo); jpeg_destroy_decompress(&cinfo); /* * release memory buffer */ free(buffer); /* * close jpeg inputing file */ fclose(infile); /* * unmap framebuffer's shared memory */ fb_munmap(data, screensize); /* * close framebuffer device */ fb_close(fbdev); return (0); } void usage(char *msg) { fprintf(stderr, "%s/n", msg); printf("Usage: fv some-jpeg-file.jpg/n"); } /* * open framebuffer device. * return positive file descriptor if success, * else return -1. */ int fb_open(char *fb_device) { int fd; if ((fd = open(fb_device, O_RDWR)) < 0) { perror(__func__); return (-1); } return (fd); } /* * get framebuffer's width,height,and depth. * return 0 if success, else return -1. */ int fb_stat(int fd, int *width, int *height, int *depth) { struct fb_fix_screeninfo fb_finfo; struct fb_var_screeninfo fb_vinfo; if (ioctl(fd, FBIOGET_FSCREENINFO, &fb_finfo)) { perror(__func__); return (-1); } if (ioctl(fd, FBIOGET_VSCREENINFO, &fb_vinfo)) { perror(__func__); return (-1); } *width = fb_vinfo.xres; *height = fb_vinfo.yres; *depth = fb_vinfo.bits_per_pixel; return (0); } /* * map shared memory to framebuffer device. * return maped memory if success, * else return -1, as mmap dose. */ void * fb_mmap(int fd, unsigned int screensize) { void * fbmem; if ((fbmem = mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0)) == MAP_FAILED) { perror(__func__); return (void *) (-1); } return (fbmem); } /* * unmap map memory for framebuffer device. */ int fb_munmap(void *start, size_t length) { return (munmap(start, length)); } /* * close framebuffer device */ int fb_close(int fd) { return (close(fd)); }

编译，运行显示如下：

![](http://hi.csdn.net/attachment/201008/21/0_128238596122Ps.gif)