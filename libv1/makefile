TARGET := libuser_test.1.so
CC := gcc
CFLAGS := -fPIC -Wall
LDFLAGS := -shared
LIBVERSION := 1.0
all: $(TARGET)
	$(CC) $(CFLAGS) -o $(TARGET) user_test.c $(LDFLAGS) -Wl,-soname,libuser_test.so.$(LIBVERSION)
clean:
	rm -f $(TARGET)
