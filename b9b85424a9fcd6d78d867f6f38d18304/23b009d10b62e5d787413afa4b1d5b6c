#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main(int argc, char **argv) {
    if (argc>1) {
        gid_t gid = getegid();
        setresgid(gid, gid, gid);
        printf("Good thing you don't have /bin/sh");
        printf("\nGood luck getting a shell.\n");
        system("echo You Lose!\n");
        char buf[24];
        strcpy(buf,argv[1]);
        return 0;
    }
    else {
        return 0;
    }
}
