# Smart-Women-Security-Device
#include<pic.h>
#include"delay.c"
#include"lcd_driver.c"
#include"adc_driver.c"
#include"uart_driver.c"

__CONFIG(0x1932);

static int cnt;							// Timer1 count
static char rate_cnt,hrt_rate,flag;		// Pulse count from sensor

void main()
{
	TRISB = 0xFF;	//Set Port B  input to use external interrupt

	/* Configuring the External Interrupt */
	INTEDG = 0; 	// Interrupt edge select (1->Rising edge, 0->falling edge)
	INTE = 1;		// Enables 	the external interrupt

	/* Configuring Timer1 in timer mode */
	T1CON = 0x31;	// Enables T1 for internal clock prescaleed to 1:8,
	TMR1IE = 1;		// Enables Timer1 interrupt 
	PEIE = 1;		// Enables all peripheral interrupts
	GIE = 1;		// Enable the global interrupt 

setup_lcd_port();
lcd_init();
adc_init();
uart_init(9600,16);	

unsigned int temp;

lcd_goto_pos(1);
lcd_puts("Stop Voilence   ");
lcd_goto_pos(17);
lcd_puts("Against Women   ");
Delay(1);
uart_putc(0x2a);
Delay(2);
uart_puts("Stop Voilence Against Women");
uart_putc(0x23);
Delay(15);
lcd_clrscr();

while(1)
{
adc_channel(0);
temp=adc_res();
lcd_goto_pos(17);
lcd_puts("Temp:   C");
temp=temp*48/100;
lcd_goto_pos(22);
lcd_putn(temp);
lcd_goto_pos(24);
lcd_putc(223);

		DelayMs(100);			// Just a bit slow
		lcd_goto_pos(12);
		lcd_putn(hrt_rate);		
		if(flag)
			{
			lcd_clrscr();
			lcd_goto_pos(1);
			lcd_puts("Heart Rate:   ");
			flag = 0;
			}

uart_putc(0x2a);
Delay(2);
uart_puts("T:");
uart_putn(temp);
uart_putc(0x5F);

uart_puts("HR:");
uart_putn(hrt_rate);
uart_putc(0x23);
Delay(5);

if(temp>40)
{
uart_putc(0x2a);
Delay(2);
uart_puts("Body temp is High");
uart_putc(0x23);
Delay(15);
uart_putc(0x2a);
Delay(2);
uart_puts("Latitude: 13.501492 ");
uart_puts("longitude: 80.076297 ");
uart_putc(0x23);
Delay(15);
}

if(hrt_rate>96 && hrt_rate<76)
{
uart_putc(0x2a);
Delay(2);
uart_puts("Pulse is abnormal");
uart_putc(0x23);
Delay(15);
uart_putc(0x2a);
Delay(2);
uart_puts("Latitude: 13.049565 ");
uart_puts("longitude: 80.075267 ");
uart_putc(0x23);
Delay(15);
}

if(RB4==1)
{
lcd_goto_pos(32);
lcd_puts("1");
uart_putc(0x2a);
Delay(2);
uart_puts("Intruder Detected");
uart_putc(0x23);
Delay(15);
}
}
}	
/*------------------------------------------------------------*/
/* Interrupt Service Routine (ISR) */

static void interrupt
isr(void)		
{

/* ISR to create one minute timing interrupt using Timer1 */
if(TMR1IF)		// Interrupt occurs every 0.1 secs
{

TMR1ON = 0;
TMR1L = 0xb0;	//Reloading the Timer1 to 15536 to create 0.1 sec
TMR1H = 0x3c;
TMR1ON = 1;

cnt++;			// Increments every 0.1 sec

	if (cnt == 600)
	{	
		hrt_rate = rate_cnt;
		cnt = 0;
		rate_cnt = 0;
		flag = 1;
	}

TMR1IF = 0;			// Clear the Timer interrupt flag
}


/* ISR for External Interrupt  (pulse sensor)  */
if(INTF)
{
	rate_cnt++;

	INTF = 0;		// Clear the external interrupt flag
}
}
/*------------------------------------------------------------*/
