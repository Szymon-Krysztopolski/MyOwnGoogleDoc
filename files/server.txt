#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netdb.h>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <time.h>

int buff_size=1024;

void childend(int signo) {wait (NULL); }

int wmess(int fd, char* buff) {
   	 int bread, btotal = 0;

  	 while (btotal < buff_size) {
     	 bread = write(fd, buff + btotal, buff_size - btotal);
   		 btotal += bread;
  	 }
   	 return btotal;
}

int rmess(int fd, char* buff) {
   	 int bread, btotal = 0;
   	 
  	 while (btotal < buff_size) {
			 bread = read(fd, buff + btotal, buff_size - btotal);
  		 btotal += bread;
  	 }
  	 return btotal;
}

int is_end(char *str){
    if(str[0] == 'E' &&
    str[1] == 'O' &&
    str[2] == 'F')
   	 return 1;
    else return 0;
}


int main(int argc, char **argv)
{
    srand(time(NULL));
    socklen_t slt;
    int sfd, cfd, on=1;
    int port=1234;
    struct sockaddr_in saddr, caddr;
    char buff[buff_size];
    
    FILE *file;
    int time_wait=100000;
    char status;
    
    signal(SIGCHLD, childend);
    saddr.sin_family=AF_INET;
    saddr.sin_addr.s_addr=INADDR_ANY;
    saddr.sin_port=htons(port);
    sfd=socket(AF_INET, SOCK_STREAM, 0);
    
    setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, (char*)&on, sizeof(on));
    bind(sfd, (struct sockaddr*)&saddr, sizeof(saddr));
    listen(sfd,10);
	
	FILE *tmp = fopen("global_file.txt", "w");fclose(tmp);
    
    while(1)
    {
   	 slt=sizeof(caddr);
   	 cfd=accept(sfd,(struct sockaddr*)&caddr, &slt);
   	 
   	 if(!fork()){
   		 close(sfd);
   		 printf("new connection: %s\n", inet_ntoa((struct in_addr)caddr.sin_addr));
   		 while(1) //petla glowna
   		 {
   			 memset(buff,0,buff_size);
   			 read(cfd,buff,sizeof(buff));
   			 printf("\nb: %s\n",buff);
   			 status=buff[0];
   			 if(status=='\x0') {
   				 printf("user disconnected\n");
   				 break;
   			 }
   			 
   			 printf("start sending file\n");
   			 file=fopen("global_file.txt","r");
   			 while(fgets(buff,buff_size,file) != NULL)
   			 {
   				 printf("The line is: %s",buff);
   				 wmess(cfd,buff);
   				 usleep(time_wait); //czasem server gubi pakiety i wysyla dziwne wiadomosci, usleep rozwiazuje te problemy
   			 }
   			 
   			 wmess(cfd,"EOF\n");
   			 usleep(time_wait);
   			 fclose(file);
   			 
   			 if(status=='w'){
   				 file=fopen("global_file.txt","w");
   				 while(1){
   					 memset(buff,0,buff_size);
   					 rmess(cfd,buff);
   					 
   					 if(is_end(buff)){
   						 //fputs("\n",file);
   						 break;
   					 } else {
   						 fputs(buff,file);
   					 }
   				 }
   				 fclose(file);
   			 }
   		 }
   		 close(cfd);
   		 exit(EXIT_SUCCESS);
   	 }
   	 close(cfd);
    }
    close(sfd);
    return EXIT_SUCCESS;
}
