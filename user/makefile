TARGET := user_program
CC := gcc
LDFLAGS := -luser_test

all: $(TARGET)
	$(CC) -o $(TARGET) main.c $(LDFLAGS)
clean:
	rm -f $(TARGET)