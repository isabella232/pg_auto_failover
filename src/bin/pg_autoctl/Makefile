# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the PostgreSQL License.

PG_AUTOCTL = ./pg_autoctl

SRC_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))

DEPDIR = $(SRC_DIR)/.deps

INCLUDES  = $(patsubst ${SRC_DIR}%.h,%.h,$(wildcard ${SRC_DIR}*.h))

SRC   = $(patsubst ${SRC_DIR}%.c,%.c,$(wildcard ${SRC_DIR}*.c))
OBJS  = $(patsubst %.c,%.o,$(SRC))
OBJS += lib-log.o lib-commandline.o lib-parson.o lib-snprintf.o lib-strerror.o

PG_CONFIG ?= pg_config
BINDIR    ?= $(shell $(PG_CONFIG) --bindir)

PG_SNPRINTF     = $(wildcard ${SRC_DIR}../lib/pg/snprintf.*)
LOG_SRC         = $(wildcard ${SRC_DIR}../lib/log/src/log.*)
COMMANDLINE_SRC = $(wildcard ${SRC_DIR}../lib/subcommands.c/commandline.*)
PARSON_SRC      = $(wildcard ${SRC_DIR}../lib/parson/parson.*)

COMMON_LIBS  = -I${SRC_DIR}../lib/pg
COMMON_LIBS += -I${SRC_DIR}../lib/log/src/
COMMON_LIBS += -I${SRC_DIR}../lib/subcommands.c/
COMMON_LIBS += -I${SRC_DIR}../lib/libs/
COMMON_LIBS += -I${SRC_DIR}../lib/parson/

CC = $(shell $(PG_CONFIG) --cc)

DEFAULT_CFLAGS = -std=c99 -D_GNU_SOURCE -g
DEFAULT_CFLAGS += -I $(shell $(PG_CONFIG) --includedir)
DEFAULT_CFLAGS += -I $(shell $(PG_CONFIG) --includedir-server)
DEFAULT_CFLAGS += -I $(shell $(PG_CONFIG) --pkgincludedir)/internal
DEFAULT_CFLAGS += $(shell $(PG_CONFIG) --cflags)
DEFAULT_CFLAGS += -Wformat
DEFAULT_CFLAGS += -Wall
DEFAULT_CFLAGS += -Werror=implicit-int
DEFAULT_CFLAGS += -Werror=implicit-function-declaration
DEFAULT_CFLAGS += -Werror=return-type
DEFAULT_CFLAGS += -Wno-declaration-after-statement

# Needed for OSX
DEFAULT_CFLAGS += -Wno-missing-braces
DEFAULT_CFLAGS += $(COMMON_LIBS)

ifdef USE_SECURITY_FLAGS
# Flags taken from: https://liquid.microsoft.com/Web/Object/Read/ms.security/Requirements/Microsoft.Security.SystemsADM.10203#guide
SECURITY_CFLAGS=-fstack-protector-strong -D_FORTIFY_SOURCE=2 -O2 -z noexecstack -fpie -Wl,-pie -Wl,-z,relro -Wl,-z,now -Wformat -Wformat-security -Werror=format-security
DEFAULT_CFLAGS += $(SECURITY_CFLAGS)
endif

override CFLAGS := $(DEFAULT_CFLAGS) $(CFLAGS)

LIBS  = -L $(shell $(PG_CONFIG) --pkglibdir)
LIBS += -L $(shell $(PG_CONFIG) --libdir)
LIBS += $(shell $(PG_CONFIG) --ldflags)
LIBS += $(shell $(PG_CONFIG) --libs)
LIBS += -lpq
LIBS += -lncurses

all: $(PG_AUTOCTL) ;

# Based on Postgres Makefile for automatic dependency generation
# https://github.com/postgres/postgres/blob/1933ae629e7b706c6c23673a381e778819db307d/src/Makefile.global.in#L890-L924
%.o : %.c
	@if test ! -d $(DEPDIR); then mkdir -p $(DEPDIR); fi
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -o $@ $<

Po_files := $(wildcard $(DEPDIR)/*.Po)
ifneq (,$(Po_files))
include $(Po_files)
endif


$(PG_AUTOCTL): $(OBJS) $(INCLUDES)
	$(CC) $(CFLAGS) $(OBJS) $(LDFLAGS) $(LIBS) -o $@

lib-snprintf.o: $(PG_SNPRINTF)
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -MT$@ -o $@ ${SRC_DIR}../lib/pg/snprintf.c

lib-strerror.o: $(PG_SNPRINTF)
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -MT$@ -o $@ ${SRC_DIR}../lib/pg/strerror.c

lib-log.o: $(LOG_SRC)
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -MT$@ -o $@ ${SRC_DIR}../lib/log/src/log.c

lib-commandline.o: $(COMMANDLINE_SRC)
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -MT$@ -o $@ ${SRC_DIR}../lib/subcommands.c/commandline.c

lib-parson.o: $(PARSON_SRC)
	$(CC) $(CFLAGS) -c -MMD -MP -MF$(DEPDIR)/$(*F).Po -MT$@ -o $@ ${SRC_DIR}../lib/parson/parson.c

clean:
	rm -f $(OBJS) $(PG_AUTOCTL)
	rm -rf $(DEPDIR)

install: $(PG_AUTOCTL)
	install -d $(DESTDIR)$(BINDIR)
	install -m 0755 $(PG_AUTOCTL) $(DESTDIR)$(BINDIR)



.PHONY: all monitor clean
