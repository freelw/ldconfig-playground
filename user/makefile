TARGET := user_program
CC := gcc
LDFLAGS := -luser_test

all: $(TARGET)

$(TARGET): main.c
	$(CC) -o $(TARGET) main.c $(LDFLAGS)
clean:
	rm -f $(TARGET)