# budva32

Telegram-Forwarder (UNIX-way)

## How to install tdlib (for use w/o docker)

For Ubuntu 18.04

```
$ sudo apt-get install build-essential gperf ccache zlib1g-dev libssl-dev libreadline-dev
```

Or use [TDLib build instructions](https://tdlib.github.io/td/build.html)

## .env

[Register an application](https://my.telegram.org/apps) to obtain an api_id and api_hash

```
BUDVA32_API_ID=1234567
BUDVA32_API_HASH=XXXXXXXX
BUDVA32_PHONENUMBER=78901234567
BUDVA32_PORT=4004
```

## First start for Telegram auth via web

http://localhost:4004

<!-- ## Old variants for Telegram auth (draft)

from console:

```
$ go run .
```

or via docker:

```
$ make
$ make up
$ docker attach telegram-forwarder
```

but then we have problem with permissions (may be need docker rootless mode?):

```
$ sudo chmod -R 777 ./tdata
``` -->

## config.yml example

```yml
Others:
  -4444:
    SourceTitle: "*Channel Name*☝️" # for SendCopy (with markdown)
Reports:
  To: [
      -2222,
      -4321,
      -8888,
    ]
  Template: "За *24 часа* отобрал: *%d* из *%d* 😎" # (with markdown)
Forwards:
	- From: -1111
		To: [-2222]
		WoSendCopy: true
	- From: -1234
		To: [-4321, -8888]
		Other: -4444
		# WithEdited: true # deprecated
		Exclude: 'Крамер|#УТРЕННИЙ_ОБЗОР'
		Include: '#ARK|#Идеи_покупок|#ОТЧЕТЫ'
		IncludeSubmatch:
			- Regexp: '(^|[^A-Z])\$([A-Z]+)'
				Group: 2
				Match: ['F', 'GM', 'TSLA']
```

## Get chat list with limit (optional)

http://localhost:4004?limit=10

<!-- ## About ReplyToMessageId

До боли простое решение по копированию сообщений с историей изменений. Если копируемое сообщение было отредактировано, то копирование выполняется в новое сообщение с отсылкой на предыдущее, благодаря механизму ответов.

![](assets/image1.jpg)

И можно получить отдельно всю историю в просмотре ответов (работает только для группы, но не для канала).

![](assets/image2.jpg) -->

## Examples for go-tdlib

```go
func getMessageLink(srcChatId, srcMessageId int) {
	src, err := tdlibClient.GetMessage(&client.GetMessageRequest{
		ChatId:    int64(srcChatId),
		MessageId: int64(srcMessageId),
	})
	if err != nil {
		fmt.Print("GetMessage src ", err)
	} else {
		messageLink, err := tdlibClient.GetMessageLink(&client.GetMessageLinkRequest{
			ChatId:     src.ChatId,
			MessageId:  src.Id,
			ForAlbum:   src.MediaAlbumId != 0,
			ForComment: false,
		})
		if err != nil {
			fmt.Print("GetMessageLink ", err)
		} else {
			fmt.Print(messageLink.Link)
		}
	}
}

// How to use update?

	for update := range listener.Updates {
		if update.GetClass() == client.ClassUpdate {
			if updateNewMessage, ok := update.(*client.UpdateNewMessage); ok {
				//
			}
		}
	}

// etc
// https://github.com/zelenin/go-tdlib/blob/ec36320d03ff5c891bb45be1c14317c195eeadb9/client/type.go#L1028-L1108

// How to use markdown?

	formattedText, err := tdlibClient.ParseTextEntities(&client.ParseTextEntitiesRequest{
		Text: "*bold* _italic_ `code`",
		ParseMode: &client.TextParseModeMarkdown{
			Version: 2,
		},
	})
	if err != nil {
		log.Print(err)
	} else {
		log.Printf("%#v", formattedText)
	}

```

## Inspired by

- [marperia/fwdbot](https://github.com/marperia/fwdbot)
- [wcsiu/telegram-client-demo](https://github.com/wcsiu/telegram-client-demo) + [article](https://wcsiu.github.io/2020/12/26/create-a-telegram-client-in-go-with-docker.html)
- [Создание и развертывание ретранслятора Telegram каналов, используя Python и Heroku](https://vc.ru/dev/158757-sozdanie-i-razvertyvanie-retranslyatora-telegram-kanalov-ispolzuya-python-i-heroku)
