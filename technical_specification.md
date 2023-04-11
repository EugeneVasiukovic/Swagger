### PROXY-SERVER:
PROXY server будет состоять из 3 endpoints, данные endpoints, в свою очередь, будут обращаться к веб-сервисам и возвращать информацию HTTP-клиенту.

**Сервер располагается на хосте (URL and PORT):** localhost:1425  
**Endpoints:** /join_course, /telegram_group,  /screening_info

1) **/join_course**  принимает информацию методом **`POST`**.  
- Проводит валидацию    
	*email: (наличие @, домена электронной почты, отсутствие спецсимволов кроме "_" и ".");  
	*telegram_nick:(наличие @, отсутствие спецсимволов кроме "_" и ".");  
	*linklinkidin:(количество символов от 0 до 80)  
- Перенаправляет запрос на Веб-сервис по адресу https://localhost:1426/WS2/join_course_QA .

`Request` должен передавать на сервер следующую информацию в body raw JSON:  
{  
"email": "string",   
"telegram_nick": "string",   
"link_linkidin": "string"   
}   

`Response` вернет заполненные данные и присвоет id пользователю:  
{"1":  
	{  
	"email": "string",  
	"telegram_nick": "string",  
	"link_linkidin": "string"  
	}  
}  

2) **/telegram_group** принимает информацию методом **`POST`**  
- Проводит валидацию 
	telegram_nick:(наличие @, отсутствие спецсимволов кроме "_" и ".");  
- Проводит аутентификацию telegram_nick в БД, если telegram_nick отсутствует, выводит сообщение "данного пользователя не обнаружено"  
- Перенаправляет запрос на Веб-сервис по адресу https://localhost:1428/WS2/telegram_group_QA

- `Request` должен передавать на сервер следующую информацию в body raw JSON:  
{  
"telegram_nick": "string"  
}  

- `Response` вернет JSON в котором храниться массив данных:  
{ "qa_group_curse":  
		[Employment,  
		Vadim_QA_Padawans_Channel,  
		QA_Python_Course_,
		Practice waiting room,
		№_qa_group,
		English Practice by A.Kryshnev & V.Ksendzov]
  "info_message": "если вы не добавлены в какую либо из представленных групп напишите @iren_melnychenko"  
}  


3) **/screening_info** принимает информацию методом **`GET`**
- Перенаправляет запрос на Web-сервис по адресу https://localhost:1427/WS1/screening_info_QA.  
- Данный endpoint не имеет параметров в `Request`. 
- `Response` возвращает JSON в котором хранятся данные:  
{  
"HomeWorks_list":  
	{  
	"Topic1":"Terminal_Linux',  
	"Topic2":"Git",  
	"Topic3":"SQL",  
	"Topic4":"Postman",  
	"Topic5":"ClientServer_Theory",  
	"Topic6":"MobilTesting",  
	"scrining_link": "https://docs.google.com/spreadsheets/d/1_aa1_hpEVJyhglhSOgZpeuT7hhURRo5EQETqobnAfuU/edit#gid=2131563121"  
	}  
}  


### WEB-SERVICE 1:
**Сервер располагается на хосте (URL and PORT):** localhost:1427/WS1  
**Endpoint:** /screening_info_QA  
- Принимает информацию методом **`GET`**  
- Отправляет `Response`  на PROXY-сервер(localhost:1425/screening_info) в формате JSON:  
{  
"HomeWorks_list":  
{  
"Topic1":"Terminal_Linux',  
"Topic2":"Git",  
"Topic3":"SQL",  
"Topic4":"Postman",  
"Topic5":"ClientServer_Theory",  
"Topic6":"MobilTesting",  
"scrining_link": "https://docs.google.com/spreadsheets/d/1_aa1_hpEVJyhglhSOgZpeuT7hhURRo5EQETqobnAfuU/edit#gid=2131563121"  
}  
}  

### WEB-SERVICE 2:
**Сервер распологается на хосте (URL and PORT):** localhost:1428/WS2  
**Endpoint:** 	/telegram_group_QA  
- Принимает информацию методом **`POST`**  
- Происходит аутентификация telegram_nick в БД, через редирект на https://localhost:1426/WS2/join_course_QA, 
- после успешной аутентификации отправляет `Response`  на PROXY-сервер (*localhost:1425/telegram_group*) в формате JSON:  
{ "qa_group_course":  
[  
Employment,  
Vadim_QA_Padawans_Channel,  
QA_Python_Course,  
Practice waiting room,  
№_qa_group,  
English Practice by A.Kryshnev & V.Ksendzov]  
"info_message": "если вы не добавлены в какую либо из представленных групп напишите @iren_melnychenko"  
}  
- Если аутентификация по "telegram_nick" не пройдена, выводится сообщение "данного пользователя не обнаружено".  


**Сервер распологается на хосте (URL and PORT):** localhost:1426/WS2  
**Endpoints:** /join_course_QA  
- Принимает информацию методом **`POST`**  
- Создает сущность в БД  
- Отправляет `Response` на PROXY-сервер(*localhost:1425/join_course*) в формате JSON:  
{"1":  
{    
"email": "string",   
"telegram_nick": "string",  
"link_linkidin": "string"  
}  
}  

