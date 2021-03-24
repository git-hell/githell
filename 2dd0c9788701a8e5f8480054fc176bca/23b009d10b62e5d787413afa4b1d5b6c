#include<stdio.h>
#include<string.h>
int main(int argc, char **argv) {
    putenv("PATH=");
    printf("I've broken up my system call!\n");
    printf("You think I've included what you need for this? You wish\n");
    char user_buf[64]= "";
    if (argc > 1) {
        strcpy(user_buf,argv[1]);
    }
    else {
        printf("usage: ./overflow [input]\n");
        return 0;
        }
    char buf1[10] = "/b";
    char buf2[8] = "in/";
    char buf3[5] = "date";
    strcat(buf2,buf3);
    strcat(buf1,buf2);
    system(buf1);
    printf("Aren't these string functions wonderful?\n");
    return 0;
}
