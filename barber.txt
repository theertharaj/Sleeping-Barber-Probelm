#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
#include<errno.h>
#include<sys/ipc.h>
#include<semaphore.h>
 
int t=0,N=0; 
void b();
void c();
sem_t mutex,customers,barbers; 
time_t end_time;
int count=0;

int main(void)
{
	printf("Enter the number of Working Hours of Barber(in Seconds)\n");
        scanf("%d",&t);
	printf("Enter the number of available chairs in the waiting room..\n");
	scanf("%d",&N);
	pthread_t id1,id2;
	int status=0;
	end_time=time(NULL)+t;
 
	sem_init(&mutex,0,1);
	sem_init(&customers,0,0);
	sem_init(&barbers,0,1);
 
	pthread_create(&id1,NULL,(void *)b,NULL);	
	pthread_create(&id2,NULL,(void *)c,NULL); 
	pthread_join(id2,NULL);
	pthread_join(id1,NULL);
 
	exit(0);
}
 
void b()
{
	int a=0,j=0;
	while(time(NULL)<end_time || count>0)
	{
		
		if(count==1&&j==0)
		{
			printf("Barber: sleeping, Customer wakes him and get served \n");
			j=1;
				
		}
		       
		sem_wait(&mutex);
		count--;
		printf("Barber:finished serving customer\n\t Number of Customers waiting :%d\n",count);
		sem_post(&mutex);
		sem_post(&barbers);
		if(time(NULL)>end_time && count>=0)
		{
			a++;
			
		}
		if(a==1)
		{
			
			printf("\nBarber:Sorry!!! Working hours are over!!!Shop will be closed-No more new customers,serving remaining customers\n\n");
			continue;	
		}
		sleep(3);       
	}
	
}
 
void c()
{
	while(time(NULL)<end_time)
	{
				
		if(count>=N)
		{	
			printf("Customer:All Chairs are FILLED! Customer go back.\n");
			sleep(3);
			continue;
		}	
			sem_wait(&mutex);
		
			count++;
			printf("Customer:Arrived,Chair is available. Takes the seat\n\t Number of Customers waiting :%d\n",count);
			sem_post(&mutex);
			sem_post(&customers);
			sem_post(&mutex);
		
		sleep(1);
	}
}

