#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* run this program using the console pauser or add your own getch, system("pause") or input loop */
struct player {      //创建链表 
	int uid; 
	char name[30];
	char password[8];
	struct player *next;
};
struct playerlist {
	struct player *head;
	int length;
};
void initlink(struct playerlist *link) {
	link -> head = NULL;
	link -> length = 0;
}
struct player * getelement(struct playerlist *link, int pos) {   //登录 
	if(pos < 0 || pos > link -> length) {
		printf("查无此人\n"); 
		return NULL;
	}
	struct player *p = link -> head;
	int i = 0;
	for(i = 0; i < pos; i++) {
		p = p -> next;
	}
	return p;
}
int insert(struct playerlist *link, int pos, struct player* player) {  //初始化需要 
	if(pos < 0 || pos > link -> length) {
		return NULL;
	}
	if(pos == 0) {  
		player -> next = link -> head;
		link -> head = player;
	}
	else { 
		struct player *prev = getelement(link,pos - 1);
		player -> next = prev -> next;
		prev -> next = player;
	}
	link -> length++;  //更新链表长度 
	return 1;
}


int insert_after(struct playerlist *link, int pos, struct player* player) {  //插入需要 
	int i;
	if(pos < 0 || pos > link -> length) {
		return NULL;
	}
	if(pos == 0) { //表头插入 
		player -> next = link -> head;
		link -> head = player;
	}
	else {  //非表头插入 
		struct player *prev = getelement(link,pos - 1);
		player -> next = prev -> next;
		prev -> next = player;
	}
	struct player*cur = link -> head;
	do {
		cur -> uid += 1;
		cur = cur -> next;
	}while(cur -> next != NULL);
	player -> uid = link -> length - pos + 1;
	printf("请输入姓名:");
	scanf("%s",player->name);
	link -> length++;  //更新链表长度 
	return 1;
}
int showmenu() {        //菜单 
	printf("\t\t|========================请选择您要进行的操作=======================|\t\n");
	printf("\t\t|=====\t\t\t\t注册1\t\t\t\t====|\t\n");
	printf("\t\t|=====\t\t\t\t登录2\t\t\t\t====|\t\n");
	printf("\t\t|=====\t\t\t\t修改3\t\t\t\t====|\t\n");
	printf("\t\t|=====\t\t\t\t插入4\t\t\t\t====|\t\n");
	printf("\t\t|=====\t\t\t\t打印5\t\t\t\t====|\t\n");
	printf("\t\t|=====\t\t\t\t退出6\t\t\t\t====|\t\n");
	printf("\t\t|===================================================================|\t\n");
	int result=0;
	printf("请选择您要进行的操作:");
	scanf("%d", &result);
	return result;
}
void showmenuresult(int result) {          //选择操作 
	switch(result) {
		case 1:
			printf("您选择了1注册操作\n");
			break;
		case 2:
			printf("您选择了2登录操作\n");
			break;
		case 3:
			printf("您选择了3修改操作\n");
			break;
		case 4:
			printf("您选择了4插入操作\n");
			break;
		case 5:
			printf("您选择了5打印操作\n");
			break;
		default:
			printf("未识别的操作\n");
			break;
	}
}
void addstuinfo(struct playerlist* link) {                  //注册
	struct player *player=(struct player*)malloc(sizeof(struct player));
	player -> next = NULL;
	
	player -> uid = link -> length + 1;
	
	printf("请输入姓名:");
	scanf("%s",player->name);
	
	insert(link,0,player); 
}

void savestuinfo(struct playerlist*link) {               //保存 
	if(link -> length == 0) {
		printf("没有玩家数据\n");
		return; 
	}
	else {
		FILE *fp = fopen("player.txt","w");
		if(fp == NULL) {
			printf("无法打开player.txt文件\n");
			return NULL;
		}
		struct player *cur = link -> head;
		do {
			fprintf(fp,"%d %s\n", cur -> uid, cur -> name);
			cur = cur -> next;
		} 
		while(cur != NULL);
		fclose(fp);
		printf("保存成功\n"); 
	}
}
void readstuinfo(struct playerlist*link) {            //读取 
	FILE*fp=fopen("player.txt", "r");
	
	int uid;
	char name[30];
	
	while(fscanf(fp,"%d%s", &uid, name) != EOF) {
		struct player*player=(struct player*)malloc(sizeof(struct player));
		player -> uid = uid;
		strcpy(player -> name, name);
		insert(link, link -> length, player);
		player=(struct player*)malloc(sizeof(struct player));
	}
	fclose(fp);
}
void showstuscores(struct playerlist*link) {                   //打印基本信息 
	struct player*cur = link -> head;
	if(link -> length == 0) {
		printf("无玩家信息\n");
		return;
	}
	else {
		printf("当前共%d位玩家,各位玩家数据如下:\n", link->length);
		do {
			printf("uid:%d,姓名:%s\n", cur -> uid, cur -> name);
			cur = cur -> next;
		}
		while(cur != NULL);
	}
}
void freeLink(struct playerlist*link) {
	if(link -> length == 0) {
		return;
	}
	struct player*p = link -> head -> next;
	while(p) {
		link -> head -> next = p -> next;
		free(p);
		p = link -> head -> next;
	}
	link -> length = 0;
}
int main(int argc, char *argv[]) {
	int pos;
	int result = showmenu();
	struct playerlist link;
	struct player player;
	initlink(&link);
	
	freeLink(&link);  //读取保存在本机上的信息 
	initlink(&link);
	readstuinfo(&link);
	
	while(result != 6) {
		showmenuresult(result);
		switch(result) {
			case 1:
				addstuinfo(&link);
				break;
			case 2:
				printf("请输入uid："); 
				scanf("%d", &pos);
				if(getelement(&link, link.length - pos) != NULL){
					printf("uid:%d,姓名:%s\n", getelement(&link, link.length - pos) -> uid, getelement(&link, link.length - pos) -> name);  //找到 后的打印 	
				} 
				break;
			case 4:
				printf("请输入插入后的uid：");
				scanf("%d", &pos);
				insert_after(&link, link.length - pos + 1, &player);
				break;
			case 5:
				showstuscores(&link);
				break;
		}
		result = showmenu();
	}
	savestuinfo(&link);   //退出前的保存 
	printf("您选择了退出操作\n");
	freeLink(&link);
	return 0;
}
