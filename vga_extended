//
// VGA 64 demo - using a COG C based VGA driver.
// This is a very basic translation of Kwabena W. Agyeman's
// VGA64 6 Bits Per Pixel demo to C. I haven't included all
// of the methods of the original Spin object, just enough to
// get the visual demo going.
// Interested readers should get the original object; it is very
// well written and commented (unlike this code, unfortunately, which
// is just a proof-of-concept).
//
// The original is Copyright (c) 2010 Kwabena W. Agyeman
// C version is Copyright (c) 2011 Eric R. Smith
// Terms of use (MIT license) at end of file.


#define printf __simple_printf
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "vga.h"
#include "propeller.h"

volatile struct {
    unsigned stack[16];
    struct vga_param v;
} par;

/*
 * function to start up a new cog running the toggle
 * code (which we've placed in the .coguser0 section)
 */
void startvga(volatile void *parptr)
{
    extern unsigned int _load_start_vga_cog[];
    int r;
    r = cognew(_load_start_vga_cog, parptr);
    printf("started cog %d\n", r);
}

#define COLS 160
#define ROWS 120
#define PINGROUP 2

unsigned char framebuffer[ROWS*COLS];

static void
scrollfill(unsigned char color)
{
    int i;
    for (i = 0; i < ROWS; i++) {
        memcpy(framebuffer, framebuffer+COLS, COLS*(ROWS-1));
        memset(framebuffer+(COLS*(ROWS-1)), color, COLS);
    }
}

/* draw a horizontal line */
void
hline(unsigned char *frame, int x, int y, int w, unsigned char color)
{
    int i;
    frame += (y*COLS) + x;
    if (x + w > COLS)
        w = COLS - x;
    for (i = 0; i < w; i++)
        *frame++ = color;
}

/* draw a horizontal line */
void
point_xy(unsigned char *frame, int x, int y, int w, unsigned char color)
{
    int i;
    frame += (y*COLS) + x;
    if (x + w > COLS)
        w = COLS - x;
    //for (i = 0; i < w; i++)
    *frame = color;
}

void
box(unsigned char *frame, int x, int y, int w, int h, unsigned char color)
{
    int i;
    if (y + h > ROWS)
        h = ROWS - y;
    for (i = 0; i < h; i++) {
        hline(frame, x, y, w, color);
        y++;
    }
}


/* draw a horizontal line */
void
draw_line(unsigned char *frame, int x1, int y1, int x2, int y2, unsigned char color)
{
	    int delta_x =(x2 - x1);
	    // if x1 == x2, then it does not matter what we set here
	    signed char const ix =((delta_x > 0) - (delta_x < 0));
	    delta_x = abs(delta_x) << 1;
       int delta_y =(y2 - y1);
	    // if y1 == y2, then it does not matter what we set here
	    signed char const iy =((delta_y > 0) - (delta_y < 0));
	    delta_y = abs(delta_y) << 1;

	    point_xy(frame, x1, y1, 1, color );

	    if (delta_x >= delta_y)
	    {
		    // error may go below zero
		    int error= (delta_y - (delta_x >> 1));
		    while (x1 != x2)
		    {
		        if ((error >= 0) && (error || (ix > 0)))
		        {
		            error -= delta_x;
		            y1 += iy;
		        }
		        // else do nothing
		        error += delta_y;
		        x1 += ix;
		        point_xy(frame, x1, y1, 1, color);
	    	}
	        }
	        else
	        {
		    // error may go below zero
		    int error = (delta_x - (delta_y >> 1));
	    	while (y1 != y2)
		    {
		        if ((error >= 0) && (error || (iy > 0)))
		        {
		            error -= delta_y;
		            x1 += ix;
		        }
		        // else do nothing
	    	    error += delta_x;
		        y1 += iy;
		        point_xy(frame, x1, y1, 1, color);
		    }
	    }
   }

//};
//}

void
displayWait(int frames)
{
    int frameNo;
    while (frames-- > 0) {
        frameNo = par.v.frameCounter;
        do {} while (frameNo == par.v.frameCounter);
    }
}

int
main()
{
    unsigned int frequency, i, testval;
    unsigned int clkfreq = _CLKFREQ;

    _DIRA = (1<<15);
    _OUTA = 0;

    printf("vga demo starting...\n");
    par.v.displayBuffer = framebuffer;
    par.v.directionState = (0xff << (8*PINGROUP));
    par.v.videoState = 0x300000FF | (PINGROUP<<9);
    testval = (25175000 + 1600) / 4;
    frequency = 1;
    for (i = 0; i < 32; i++) {
        testval = testval << 1;
        frequency = (frequency << 1) | (frequency >> 31);
        if (testval >= clkfreq) {
            testval -= clkfreq;
            frequency++;
        }
    }
    par.v.frequencyState = frequency;
    par.v.enabled = 1;

    // start up the video cog
    memset(framebuffer, Grey, ROWS*COLS);
    startvga(&par.v);

    // now run the display
    srand(_CNT);
    for(;;) {
        int x, y, w, h;
        int color;

        //displayWait(5);
        x = rand() % COLS;
        y = rand() % ROWS;
        color = rand() % 256;
        draw_line(framebuffer, 0, 0, x, y, color);
    }
}
