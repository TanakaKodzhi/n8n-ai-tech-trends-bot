{
  "name": "Бот-ассистент «Тренды ИИ и IT за день»",
  "nodes": [
    {
      "parameters": {
        "content": "**Проект 6: Бот-ассистент «Тренды ИИ и IT за день» для Telegram**\n\n**​Суть**. Раз в день бот заходит на крупный новостной ИТ-ресурс (например, Хабр или зарубежный TechCrunch) через их открытый RSS-канал, собирает заголовки и краткое описание всех статей за последние 24 часа. Затем он отправляет этот массив данных в текстовую нейросеть, которая отбирает ТОП-3 самые важные новости для начинающего разработчика и пишет по ним ультра-краткую выжимку «человеческим языком». Готовый пост публикуется в твой личный Telegram-канал.",
        "height": 176,
        "width": 544
      },
      "type": "n8n-nodes-base.stickyNote",
      "typeVersion": 1,
      "position": [
        576,
        -112
      ],
      "id": "d0017e81-0633-4c38-8e98-37c28c47070d",
      "name": "Sticky Note"
    },
    {
      "parameters": {
        "content": "### 🛠 Инструкция по настройке перед запуском:\n\n1. **DeepSeek Chat Model**:\n   - Перейдите в узел и подключите свой API-ключ DeepSeek в разделе `Credential to connect with`.\n2. **Send a text message (Telegram)**:\n   - Создайте и привяжите `Telegram API Credential` (токен от @BotFather).\n   - В поле `Chat ID` укажите ID вашего канала/чата (для публичных каналов: `@имя_канала`, для приватных: ID формата `-100...`).\n   - Добавьте бота в канал в качестве администратора с правом публикации сообщений.\n3. **Schedule Trigger**:\n   - Проверьте часовой пояс в настройках Workflow (`Settings -> Timezone`) и настройте желаемое время запуска.\n4. **Активация**:\n   - Протестируйте цепочку вручную, затем переведите переключатель в положение **Active**.",
        "height": 340,
        "width": 420,
        "color": 6
      },
      "type": "n8n-nodes-base.stickyNote",
      "typeVersion": 1,
      "position": [
        1008,
        -112
      ],
      "id": "f5e90a12-8e11-47c3-b092-23fcd8481234",
      "name": "Deployment Guide"
    },
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "triggerAtHour": 7
            }
          ]
        }
      },
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.4,
      "position": [
        0,
        -48
      ],
      "id": "c17ac62b-fc51-48e6-af13-d14f50abb228",
      "name": "Schedule Trigger"
    },
    {
      "parameters": {
        "url": "https://habr.com/ru/rss/flows/backend/articles/",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        224,
        -48
      ],
      "id": "dc786d3d-ce56-4ac9-aabf-dcdd26de6ac8",
      "name": "HTTP Request"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "n8n-nodes-base.xml",
      "typeVersion": 1,
      "position": [
        416,
        -48
      ],
      "id": "a1597026-2ab9-49d1-ba0f-f74409003866",
      "name": "XML"
    },
    {
      "parameters": {
        "fieldToSplitOut": "rss.channel.item",
        "options": {}
      },
      "type": "n8n-nodes-base.splitOut",
      "typeVersion": 1.1,
      "position": [
        16,
        240
      ],
      "id": "04dbb115-64c2-4169-b246-278e93cb596f",
      "name": "Split Out"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 4
          },
          "conditions": [
            {
              "id": "b3a8b94d-5a5f-4ccb-8969-113e71adf37a",
              "leftValue": "={{ $json.pubDate }}",
              "rightValue": "={{ $now.minus({ hours: 24 }) }}",
              "operator": {
                "type": "dateTime",
                "operation": "after"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.filter",
      "typeVersion": 2.4,
      "position": [
        224,
        240
      ],
      "id": "db417fdf-293b-4aae-a17a-03c845466408",
      "name": "Filter"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Ты — редактор дайджеста для начинающих разработчиков.\nТвоя задача: изучить предоставленный JSON-массив статей и отобрать ровно ТОП-3 самые полезные и практически применимые новости для джуниор-разработчика.\n\nПРАВИЛА И ОГРАНИЧЕНИЯ:\n1. Опирайся СТРОГО на факты из заголовка (title) и описания (description). Запрещено выдумывать факты, которых нет в исходных статьях.\n2. Не используй рекламные штампы и клише. Пиши просто, емко и понятно: в чем суть события/инструмента и чем это полезно джуну.\n3. Ссылку (URL) бери ТОЧНО из поля \"link\" выбранного элемента.\n4. Выбери РОВНО 3 новости.\n\nЗаполни данные строго в соответствии с заданной схемой:\n- title: заголовок новости\n- link: оригинальная ссылка из поля link\n- summary: емкая выжимка статьи для джуна в 2-3 предложения\n\n---\nВХОДНЫЕ ДАННЫЕ (JSON):\n{{ JSON.stringify($json['title, category, description']) }}",
        "hasOutputParser": true,
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        640,
        96
      ],
      "id": "adecc771-907d-403d-89dd-891168d55c3d",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "aggregate": "aggregateAllItemData",
        "destinationFieldName": "title, category, description",
        "options": {}
      },
      "type": "n8n-nodes-base.aggregate",
      "typeVersion": 1,
      "position": [
        448,
        240
      ],
      "id": "893c34a8-ec41-4fd6-9c44-bc6366acef1c",
      "name": "Aggregate"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatDeepSeek",
      "typeVersion": 1,
      "position": [
        640,
        288
      ],
      "id": "a4b7f78c-0ca6-4c9e-8221-443a4856b319",
      "name": "DeepSeek Chat Model"
    },
    {
      "parameters": {
        "jsonSchemaExample": "[\n  {\n  \"title\": \"Заголовок новости\",\n  \"link\": \"https://habr.com/ru/articles/123456/\",\n  \"summary\": \"Краткая выжимка статьи для джуна в 2-3 предложения\",\n  \"pubDate\": \"Mon, 02 Nov 2026 21:34:10 GMT\"\n}\n]"
      },
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        784,
        288
      ],
      "id": "4eccbeb2-e562-4ab8-8d6e-680ef10bba1c",
      "name": "Structured Output Parser"
    },
    {
      "parameters": {
        "chatId": "YOUR_TELEGRAM_CHAT_ID",
        "text": "=🔥 <b>ТОП-3 IT-новости за сегодня</b>  \n\n1️⃣ <b>{{ $json.output[0].title }}</b>\n📋 О чём статья: {{ $json.output[0].summary }}\n👉<b><a href=\"{{ $json.output[0].link }}\">Читать далее</a></b>\n\n2️⃣ <b>{{ $json.output[1].title }}</b>\n📋 О чём статья: {{ $json.output[1].summary }}\n👉<b><a href=\"{{ $json.output[1].link }}\">Читать далее</a></b>\n\n3️⃣ <b>{{ $json.output[2].title }}</b>\n📋 О чём статья: {{ $json.output[2].summary }}\n👉<b><a href=\"{{ $json.output[2].link }}\">Читать далее</a></b>",
        "additionalFields": {
          "appendAttribution": false,
          "disable_web_page_preview": true,
          "parse_mode": "HTML"
        }
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        1008,
        96
      ],
      "id": "5de52f3e-c73a-42ae-894b-dade4ad7cf73",
      "name": "Send a text message"
    }
  ],
  "pinData": {},
  "connections": {
    "Schedule Trigger": {
      "main": [
        [
          {
            "node": "HTTP Request",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request": {
      "main": [
        [
          {
            "node": "XML",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "XML": {
      "main": [
        [
          {
            "node": "Split Out",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split Out": {
      "main": [
        [
          {
            "node": "Filter",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Filter": {
      "main": [
        [
          {
            "node": "Aggregate",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Aggregate": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "DeepSeek Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser": {
      "ai_outputParser": [
        [
          {
            "node": "AI Agent",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Send a text message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "timeSavedMode": "fixed",
    "callerPolicy": "workflowsFromSameOwner",
    "availableInMCP": false
  },
  "nodeGroups": [],
  "tags": []
}