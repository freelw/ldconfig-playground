PREFIX := libuser_test.so
MAJOR := 1
MINOR := 1
VERSION := ${MAJOR}.${MINOR}
TARGET := $(PREFIX).$(VERSION)
CC := gcc
CFLAGS := -fPIC -Wall
LDFLAGS := -shared

all: $(TARGET)

$(TARGET): user_test.c user_test.h
	$(CC) $(CFLAGS) -o $(TARGET) user_test.c $(LDFLAGS) -Wl,-soname,libuser_test.so.$(MAJOR)

clean:
	rm -f $(TARGET)

install:
	install -D -m 755 $(TARGET) /usr/local/lib/$(TARGET)
	ldconfig
	ln -sf /usr/local/lib/libuser_test.so.$(MAJOR) /usr/local/lib/$(PREFIX)
	cp user_test.h /usr/local/include/user_test.h
