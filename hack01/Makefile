CC=gcc
all: lib
lib: hacker
	$(CC) --shared -m32 Hacker.o -o libhacker.so
hacker:
	$(CC) -m32 -fpic -c Hacker.c
	
run:
	LD_PRELOAD=$(shell pwd)/libhacker.so ./docrypt $(FILE) $(KEY)
clean:
	rm Hacker.o libhacker.so


