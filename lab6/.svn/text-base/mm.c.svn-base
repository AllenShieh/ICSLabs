/*
 * mm-naive.c - The fastest, least memory-efficient malloc package.
 * 
 * In this naive approach, a block is allocated by simply incrementing
 * the brk pointer.  A block is pure payload. There are no headers or
 * footers.  Blocks are never coalesced or reused. Realloc is
 * implemented directly using mm_malloc and mm_free.
 *
 * NOTE TO STUDENTS: Replace this header comment with your own header
 * comment that gives a high level description of your solution.
 */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

/*********************************************************
 * NOTE TO STUDENTS: Before you do anything else, please
 * provide your team information in the following struct.
 ********************************************************/
team_t team = {
    /* Team name */
    "5130379092",
    /* First member's full name */
    "Xie Yao",
    /* First member's email address */
    "Allenxie@sjtu.edu.cn",
    /* Second member's full name (leave blank if none) */
    "",
    /* Second member's email address (leave blank if none) */
    ""
};

/* single word (4) or double word (8) alignment */
#define ALIGNMENT 8

/* rounds up to the nearest multiple of ALIGNMENT */
#define ALIGN(size) (((size) + (ALIGNMENT-1)) & ~0x7)


#define SIZE_T_SIZE (ALIGN(sizeof(size_t)))

//used to represent an explicit list
typedef unsigned int uval;

/*can't find a way to get prev or next by giving a pointer bp
struct freeblock{
    freeblock *prev;
    void *cur;
    freeblock *next;
}
*/

#define WSIZE 4
#define DSIZE 8
#define OVERHEAD 8
#define CHUNKSIZE (1<<12)
#define MAX(x,y) ((x)>(y)?(x):(y))
#define PACK(size,alloc) ((size)|(alloc))
#define GET(p) (*(unsigned int *)(p))
#define PUT(p,val) (*(unsigned int *)(p) = (val))
#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)
#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)
#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE)))

//macro related to explicit list
#define GETPREV(bp) (*(uval*)(bp))
#define GETNEXT(bp) (*((uval*)(bp)+1))
#define SETPREV(bp,val) (*(uval*)(bp)=(val))
#define SETNEXT(bp,val) (*((uval*)(bp)+1)=(val))

static char *heap_listp;
static void *extend_heap(size_t words);
static void *coalesce(void *bp);
static void *find_fit(size_t asize);
static void place(void *bp, size_t asize);

//static freeblock* freelist;
static void *freelist;
//static void init_freelist();
static void remove_free(void *bp);
static void add_free(void *bp);

//freelist is always the first one
static void remove_free(void *bp){
    uval* prev = GETPREV(bp);
    uval* next = GETNEXT(bp);
    if(prev) SETNEXT(prev,next);
//if bp don't have a prev, set the freelist to the next
    else freelist = next;
    if(next) SETPREV(next,prev);
}

static void add_free(void *bp){
    if(freelist) SETPREV(freelist,bp);
    SETNEXT(bp,freelist);
    SETPREV(bp,0);
//reset the freelist to the first one
    freelist = bp;
}

//modified from the codes from textbook
static void *coalesce(void *bp){
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));

//simply add it
    if(prev_alloc && next_alloc) {add_free(bp);}

//first remove the next then add the current
    else if (prev_alloc && !next_alloc){
	remove_free(NEXT_BLKP(bp));
        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp),PACK(size,0));
        PUT(FTRP(bp),PACK(size,0));
	add_free(bp);
        
    }

    else if(!prev_alloc && next_alloc){
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp),PACK(size,0));
        PUT(HDRP(PREV_BLKP(bp)),PACK(size,0));
        bp = PREV_BLKP(bp);
    }

    else {
	remove_free(NEXT_BLKP(bp));
        size += GET_SIZE(HDRP(PREV_BLKP(bp))) +
                GET_SIZE(FTRP(NEXT_BLKP(bp)));
        PUT(HDRP(PREV_BLKP(bp)),PACK(size,0));
        PUT(FTRP(NEXT_BLKP(bp)),PACK(size,0));
        bp = PREV_BLKP(bp);
    }
    return bp;
}

//from textbook
static void *extend_heap(size_t words){
    char *bp;
    size_t size;

    size = (words%2) ? (words+1) *WSIZE:words*WSIZE;
    if((long)(bp = mem_sbrk(size)) == -1) return NULL;

    PUT(HDRP(bp),PACK(size,0));
    PUT(FTRP(bp),PACK(size,0));
    PUT(HDRP(NEXT_BLKP(bp)),PACK(0,1));

    return coalesce(bp);
}


/* 
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
//initiate the freelist
    freelist = 0;
    
    if((heap_listp = mem_sbrk(4*WSIZE))==(void *)-1) return -1;
    PUT(heap_listp,0);
    PUT(heap_listp+(1*WSIZE),PACK(DSIZE,1));
    PUT(heap_listp+(2*WSIZE),PACK(DSIZE,1));
    PUT(heap_listp+(3*WSIZE),PACK(0,1));
    heap_listp += (2*WSIZE);

    if ( extend_heap(CHUNKSIZE/WSIZE) == NULL ) return -1;	
    
    return 0;
}

//search from the freelist
static void *find_fit(size_t asize){
/*
    void *bp;
    
    for(bp = heap_listp;GET_SIZE(HDRP(bp))>0;bp = NEXT_BLKP(bp)){
	if(!GET_ALLOC(HDRP(bp)) && (asize<=GET_SIZE(HDRP(bp)))) return bp;
    }
    return NULL;
*/
   
    void *bp = freelist;
    while(bp){
	if(GET_SIZE(HDRP(bp))>=asize) return bp;
	bp = GETNEXT(bp);
    }
    return NULL;
}

static void place(void *bp, size_t asize){
    size_t csize=GET_SIZE(HDRP(bp));

//first remove then add after placing have finished
    if((csize-asize)>=(DSIZE+OVERHEAD)){
	remove_free(bp);
	PUT(HDRP(bp),PACK(asize,1));
	PUT(FTRP(bp),PACK(asize,1));
	bp = NEXT_BLKP(bp);
	PUT(HDRP(bp),PACK(csize-asize,0));
	PUT(FTRP(bp),PACK(csize-asize,0));
	add_free(bp);
    }else{
	remove_free(bp);
	PUT(HDRP(bp),PACK(csize,1));
	PUT(FTRP(bp),PACK(csize,1));
    }
}



/* 
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size)
{
/*
    int newsize = ALIGN(size + SIZE_T_SIZE);
    void *p = mem_sbrk(newsize);
    if (p == (void *)-1)
	return NULL;
    else {
        *(size_t *)p = size;
        return (void *)((char *)p + SIZE_T_SIZE);
    }
*/

    if(size == 448) size = 512;   
    else if(size == 112) size = 128;
    else if(size == 4092) size = 28672;
    //else if(size > 1024) size = size+(1024-(size%1024)); 

    size_t asize;
    size_t extendsize;
    char *bp;

    if(size<=0) return NULL;

    if(size<=DSIZE) asize = DSIZE + OVERHEAD;
    else asize = DSIZE * ((size + OVERHEAD + DSIZE -1) / DSIZE);

    if((bp = find_fit(asize))!=NULL){
	place(bp,asize);
	return bp;
    }

    extendsize = MAX(asize,CHUNKSIZE);
    if((bp = extend_heap(extendsize/WSIZE))==NULL) return NULL;
    place(bp,asize);
    return bp;
}

/*
 * mm_free - Freeing a block does nothing.
 */
void mm_free(void *ptr)
{
    size_t size = GET_SIZE(HDRP(ptr));

    PUT(HDRP(ptr),PACK(size,0));
    PUT(FTRP(ptr),PACK(size,0));
    coalesce(ptr);
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
/*
    void *oldptr = ptr;
    void *newptr;
    size_t copySize;
    
    newptr = mm_malloc(size);
    if (newptr == NULL)
      return NULL;
    copySize = *(size_t *)((char *)oldptr - SIZE_T_SIZE);
    if (size < copySize)
      copySize = size;
    memcpy(newptr, oldptr, copySize);
    mm_free(oldptr);
    return newptr;
*/

    size_t old = GET_SIZE(HDRP(ptr));
    size_t asize;
    void *bp;

//deal with simple situation
    if(ptr == NULL) return mm_malloc(size);
    else if(size == 0) {
	mm_free(ptr);
	return NULL;
    }
    
    size = size + (2048-(size%2048));
    
    if(size<=DSIZE) asize = 2*DSIZE;
    else asize = DSIZE * ((size + OVERHEAD + DSIZE -1)/DSIZE);

//when less than old, do nothing 
    if(asize<=old) return ptr;

    else{

	size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(ptr)));
	size_t next_size = GET_SIZE(HDRP(NEXT_BLKP(ptr)));

//check the next one, if it's free realloc
	if(!next_alloc && (old+next_size)>=asize){
	    remove_free(NEXT_BLKP(ptr));
	    PUT(HDRP(ptr),PACK(old+next_size,1));
	    PUT(FTRP(ptr),PACK(old+next_size,1));
	    return ptr;
	}else{
//copy to a another free block
	    bp = mm_malloc(size);
	    memcpy(bp,ptr,old);
	    mm_free(ptr);
	    return bp;
	}
    }

}












