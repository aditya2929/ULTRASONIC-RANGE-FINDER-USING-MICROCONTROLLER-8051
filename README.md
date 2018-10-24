//ULTRASONIC-RANGE-FINDER-USING-MICROCONTROLLER-8051
#include<REGX51.h>
#include<intrins.h>   

#define data_port P2 
sbit trig=P3^5;
sbit echo=P3^2;
sbit rs=P1^0;
sbit rw=P1^1;
sbit en=P1^2;
 
 void delay_ms(unsigned int msec)
{
	int i,j;
	for(i=0;i<msec;i++)
    for(j=0;j<1275;j++);
}
 
void lcd_cmd(unsigned char dat) 
{
	
data_port = dat;
rs= 0;
rw=0;
en=1;
delay_ms(1);
en=0;

}
 
void lcd_data(unsigned char dat) 
{
	
data_port = dat;
rs= 1;
rw=0;
en=1;
delay_ms(1);
en=0;

}
 
void lcd_string(unsigned char *str) 
{
int i=0;
while(str[i]!='\0')
{
  lcd_data(str[i]);
  i++;
  delay_ms(1);
}
}
void lcd_number(int val,unsigned int field_length)			
	char str[5]={0,0,0,0,0};
	int i=4,j=0;
	while(val)
	{
	str[i]=val%10;
	val=val/10;
	i--;
	}
	if(field_length==-1)
		while(str[j]==0) j++;
	else
		j=5-field_length;

	if(val<0) lcd_data('-');
	for(i=j;i<5;i++)
	{
	lcd_data(48+str[i]);
	}
}
void send_pulse(void) 
{
	TH0=0x00;
	TL0=0x00; 
 trig=1;
 _nop_();_nop_();_nop_();_nop_();_nop_();	
 _nop_();_nop_();_nop_();_nop_();_nop_();
 trig=0;
    
} 
 
unsigned int get_range(void)
{
 int range=0;
 int timerval;
 
 send_pulse();
	
	while(!INT0);          
	while(INT0);        
	timerval = TH0;
	timerval = (timerval << 8) | TL0;
	TH0=0xFF;
	TL0=0xFF;
 lcd_cmd(0xc0);
 delay_ms(2);
 lcd_string("Distance:");
 lcd_cmd(0xc9);
 if(timerval<35000)   
 {
  range=timerval/59;
 }
 else
 {
	 range = 0;
 }
  lcd_number(range,3);
 lcd_string("cm");
	return range; 
 }
 
void main()
{
  lcd_cmd(0x38);
  lcd_cmd(0x0c);
  delay_ms(2);
  lcd_cmd(0x01);
  delay_ms(2);
  lcd_cmd(0x80);
  delay_ms(2);
	
  lcd_string("Range finder");
  delay_ms(20);
	
  TMOD=0x09;
  TR0=1;
  TH0=0x00;
	TL0=0x00;
 
  echo = 1;	
     
  
 while(1)
 { 
	 get_range();
   delay_ms(2);   
 }
}
