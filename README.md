1、安装镜像本身

mkdir -p baikal/{config,data}

cd baikal

sudo docker run -d --restart always --name baikal -p 13281:80 -v $(pwd)/config:/var/www/baikal/config -v $(pwd)/data:/var/www/baikal/Specific ckulka/baikal:nginx

2、配置宿主机的ng转发

更改DNS指向到主机


修改ng，增加配置文件，代理13281端口的服务到外网去


cd /etc/nginx/sites-enabled/
sudo cp pwa-demo cal-server

https://sabre.io/baikal/install/
在官网里，会有写

  rewrite ^/.well-known/caldav /dav.php redirect;
  rewrite ^/.well-known/carddav /dav.php redirect;

有两个地址是需要rewrite的，不rewrite的话，iOS是读不到地址的；

server {
    listen 80;
    server_name cal.lemonhall.me;
    # enforce https
    return 301 https://$server_name:443$request_uri;
}
server {
    listen 443 ssl http2;
    server_name cal.lemonhall.me;
    ssl_certificate /etc/letsencrypt/live/172-233-73-134.ip.linodeusercontent.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/172-233-73-134.ip.linodeusercontent.com/privkey.pem;
    location / {
        proxy_pass http://127.0.0.1:13281/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip
    }
}


reload ng的配置

sudo systemctl reload nginx

3、配置
Server Time zone 选择上海

参考资料：https://blog.csdn.net/wbsu2004/article/details/128542231

https://cal.lemonhall.me/admin/

管理员界面

到Users那边增加一个用户

lemonhall
xxxxxxx

4、然后就是开始使用了：

https://cal.lemonhall.me/dav.php

lemonhall
xxxxxxxxxx

iphone，进入设置，日历，账户，其它，添加CalDAV账户

然后输入上面的信息就算是搞定了

5、iphone稍微有点特殊

首先需要进入日历，然后点击下面，中间的【日历】，需要添加一个日历，不像android，会去寻找默认的

6、搞定了，是这个server需要加两个rewrite导致的，具体看文件可以；

  rewrite ^/.well-known/caldav /dav.php redirect;
  rewrite ^/.well-known/carddav /dav.php redirect;

7、列出所有的事件
import sys
from datetime import date
from datetime import datetime
from datetime import timedelta

import caldav

caldav_url = "https://cal.lemonhall.me/dav.php"
username = "lemonhall"
password = "xxxxxxxxxxx"

def run_examples():
    """
    Run through all the examples, one by one
    """
    ## We need a client object.
    ## The client object stores http session information, username, password, etc.
    ## As of 1.0, Initiating the client object will not cause any server communication,
    ## so the credentials aren't validated.
    ## The client object can be used as a context manager, like this:
    with caldav.DAVClient(
        url=caldav_url,
        username=username,
        password=password
    ) as client:

        ## Typically the next step is to fetch a principal object.
        ## This will cause communication with the server.
        my_principal = client.principal()

        ## The principals calendars can be fetched like this:
        calendars = my_principal.calendars()

        ## print out some information
        print_calendars_demo(calendars)

        my_new_calendar = my_principal.calendar(name="linode默认日历")
        
        # my_new_calendar = my_principal.make_calendar(
        #    name="Test calendar from caldav examples"
        # )

        ## Let's add some events to our newly created calendar
        ## add_stuff_to_calendar_demo(my_new_calendar)

        ## Let's find the stuff we just added to the calendar
        event = search_calendar_demo(my_new_calendar)

def search_calendar_demo(calendar):
    """
    some examples on how to fetch objects from the calendar
    """
    ## It should theoretically be possible to find both the events and
    ## tasks in one calendar query, but not all server implementations
    ## supports it, hence either event, todo or journal should be set
    ## to True when searching.  Here is a date search for events, with
    ## expand:
    events_fetched = calendar.search(
        start=datetime(2023, 11, 1, 1),
        end=datetime(2023, 12, 1, 1),
        event=True,
        expand=False,
    )

    ## "expand" causes the recurrences to be expanded.
    ## The yearly event will give us one object for each year
    #assert len(events_fetched) > 1

    print("here is some ical data:")
    print(events_fetched[1].data)

    return events_fetched[1]

def add_stuff_to_calendar_demo(calendar):
    """
    This demo adds some stuff to the calendar

    Unfortunately the arguments that it's possible to pass to save_* is poorly documented.
    https://github.com/python-caldav/caldav/issues/253
    """
    ## Add an event with some certain attributes
    may_event = calendar.save_event(
        dtstart=datetime(2023, 11, 14, 13),
        dtend=datetime(2023, 11, 14, 15),
        summary="Do the needful"
    )


def print_calendars_demo(calendars):
    """
    This example prints the name and URL for every calendar on the list
    """
    if calendars:
        ## Some calendar servers will include all calendars you have
        ## access to in this list, and not only the calendars owned by
        ## this principal.
        print("your principal has %i calendars:" % len(calendars))
        for c in calendars:
            print("    Name: %-36s  URL: %s" % (c.name, c.url))
    else:
        print("your principal has no calendars")

if __name__ == "__main__":
    run_examples()


8、增加一个事件：
import sys
from datetime import date
from datetime import datetime
from datetime import timedelta

import caldav

caldav_url = "https://cal.lemonhall.me/dav.php"
username = "lemonhall"
password = "xxxxxxx"

def run_examples():
    with caldav.DAVClient(
        url=caldav_url,
        username=username,
        password=password
    ) as client:

        ## Typically the next step is to fetch a principal object.
        ## This will cause communication with the server.
        my_principal = client.principal()

        ## The principals calendars can be fetched like this:
        calendars = my_principal.calendars()

        ## print out some information
        print_calendars_demo(calendars)

        my_new_calendar = my_principal.calendar(name="linode默认日历")
        
        # my_new_calendar = my_principal.make_calendar(
        #    name="Test calendar from caldav examples"
        # )

        ## Let's add some events to our newly created calendar
        add_stuff_to_calendar_demo(my_new_calendar)

        ## Let's find the stuff we just added to the calendar
        # event = search_calendar_demo(my_new_calendar)

def add_stuff_to_calendar_demo(calendar):
    """
    This demo adds some stuff to the calendar

    Unfortunately the arguments that it's possible to pass to save_* is poorly documented.
    https://github.com/python-caldav/caldav/issues/253
    """
    ## Add an event with some certain attributes
    may_event = calendar.save_event(
        dtstart=datetime(2023, 11, 14, 13),
        dtend=datetime(2023, 11, 14, 15),
        summary="测试用程序加一个事件"
    )


def print_calendars_demo(calendars):
    """
    This example prints the name and URL for every calendar on the list
    """
    if calendars:
        ## Some calendar servers will include all calendars you have
        ## access to in this list, and not only the calendars owned by
        ## this principal.
        print("your principal has %i calendars:" % len(calendars))
        for c in calendars:
            print("    Name: %-36s  URL: %s" % (c.name, c.url))
    else:
        print("your principal has no calendars")

if __name__ == "__main__":
    run_examples()

9、读取excel并同步给服务器的程序

import xlrd
from datetime import datetime, timedelta
import sys
from datetime import date
from datetime import datetime
from datetime import timedelta

import caldav

caldav_url = "https://cal.lemonhall.me/dav.php"
username = "lemonhall"
password = "xxxxxxxx"
client = caldav.DAVClient(url=caldav_url,username=username,password=password)


def print_calendars_demo(calendars):
    """
    This example prints the name and URL for every calendar on the list
    """
    if calendars:
        ## Some calendar servers will include all calendars you have
        ## access to in this list, and not only the calendars owned by
        ## this principal.
        print("your principal has %i calendars:" % len(calendars))
        for c in calendars:
            print("    Name: %-36s  URL: %s" % (c.name, c.url))
    else:
        print("your principal has no calendars")


## Typically the next step is to fetch a principal object.
## This will cause communication with the server.
my_principal = client.principal()

## The principals calendars can be fetched like this:
calendars = my_principal.calendars()

## print out some information
print_calendars_demo(calendars)

my_new_calendar = my_principal.calendar(name="linode默认日历")

data = xlrd.open_workbook("111.xls")
table = data.sheets()[1]


class one_day(object):
	def __init__(self,start_row,start_col):
		self.start_row = start_row  # 实例属性
		self.start_col = start_col  # 实例属性
		date_value = table.cell_value(start_row, start_col)
		date_formatted = datetime(1900, 1, 1) + timedelta(days=int(date_value)-2)
		self.date=date_formatted
		self.set_morning()
		self.set_afternoon()

	def set_date(self,date):
		self.date=date

	def set_morning(self):
		date_value = table.cell_value(self.start_row+1, self.start_col)
		self.morning = date_value

	def set_afternoon(self):
		date_value = table.cell_value(self.start_row+3, self.start_col)
		self.afternoon = date_value

def read_one_week(start_row):
	days = [1,2,3,4,5,6,7]
	date_value = table.cell_value(start_row-1, 1)
	print(date_value)
	if date_value=="周一":
		print("OK，确实是一周，开始解析")
		for day in days:
			a_day=one_day(start_row,day)
			print("================main=================")
			print(a_day.date)

			events_fetched = my_new_calendar.search(
				start=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 0),
				end=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 23),
				event=True,
				expand=False,
		    )
			print("I will delete everything in this day:")
			for e_in_a_day in events_fetched:e_in_a_day.delete()

			print("================上午=================")
			print(a_day.morning)
			#may_event = my_new_calendar.save_event(
			#	dtstart=datetime(2023, 11, 14, 13),
			#	dtend=datetime(2023, 11, 14, 15),
			#	summary=text
			#)
			may_event = my_new_calendar.save_event(
				dtstart=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 9),
				dtend=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 12),
				summary=a_day.morning
			)

			print("================下午=================")
			print(a_day.afternoon)
			may_event = my_new_calendar.save_event(
				dtstart=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 13),
				dtend=datetime(a_day.date.year, a_day.date.month, a_day.date.day, 17),
				summary=a_day.afternoon
			)
	else:
		print("不是一个有效的周，不做任何事情")
		pass

for row in range(1,31,6):
	#每一周加6行就好了
	read_one_week(row)
	print("=================================\n\n")
