# whatever flags you want to pass to the C compiler & linker
AM_CFLAGS = --pedantic -Wall -std=c99 -O2
AM_LDFLAGS = -lm

# this lists the binaries to produce, the (non-PHONY, binary) targets in
# the previous manual Makefile
bin_PROGRAMS = hello world
hello_SOURCES = hello.c hello.h
world_SOURCES = world.c world.h
