# Кастомизация интерфейса AI чата (пакет shinychat)
На предыдущих этапах мы научились создавать «мозг» нашего ассистента: подключать языковые модели, настраивать инструменты и базы знаний. Однако для конечного пользователя важна не только логика, но и удобство взаимодействия. Пакет `shinychat` предоставляет мощный инструментарий для создания современных чат-интерфейсов в стиле ChatGPT прямо внутри Shiny-приложений.

В этом уроке мы разберем, как выйти за рамки стандартных настроек: научимся менять визуальный стиль, добавлять интерактивные подсказки, гибко управлять отображением работы инструментов и встраивать чат в сложные модульные приложения.

## Видео
<iframe width="560" height="315" src="https://www.youtube.com/embed/xxk-sxKSoII?enablejsapi=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Тайм-коды
* [**00:00**](https://youtu.be/xxk-sxKSoII?t=0) — Введение
* [**01:04**](https://youtu.be/xxk-sxKSoII?t=64) — План урока
* [**03:05**](https://youtu.be/xxk-sxKSoII?t=185) — Базовый интерфейс: кастомизация иконки и приветствие
* [**06:03**](https://youtu.be/xxk-sxKSoII?t=363) — Добавление в чат подсказок (suggestions)
* [**07:23**](https://youtu.be/xxk-sxKSoII?t=443) — Темы bslib и закрепление поля ввода внизу страницы
* [**09:33**](https://youtu.be/xxk-sxKSoII?t=573) — Расположение интерфейса чата в sidebar
* [**10:49**](https://youtu.be/xxk-sxKSoII?t=649) — Как поместить интерфейс чата в карточку (card)
* [**11:30**](https://youtu.be/xxk-sxKSoII?t=690) — Отображение в чате вызываемых моделью инструментов
* [**14:10**](https://youtu.be/xxk-sxKSoII?t=850) — Кастомизация иконки и описания инструментов
* [**17:28**](https://youtu.be/xxk-sxKSoII?t=1048) — Кастомизация вывода результата (HTML, Markdown)
* [**20:13**](https://youtu.be/xxk-sxKSoII?t=1213) — Глобальные опции управления отображением инструментов
* [**21:34**](https://youtu.be/xxk-sxKSoII?t=1294) — Восстановление и сброс диалога (chat_restore, chat_clear)
* [**24:51**](https://youtu.be/xxk-sxKSoII?t=1491) — Использование shinychat в модульных приложениях
* [**27:56**](https://youtu.be/xxk-sxKSoII?t=1676) — Заключение

## Презентация
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/hxHKOvkuaCpbgz?hostedIn=slideshare&page=upload" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

## Конспект
### Кастомизация интерфейса чата
#### Базовый интерфейс с кастомизацией иконки ассистента
Пакет `shinychat` позволяет быстро построить графический интерфейс чата, минимальный пример кода выглядит примерно так:


``` r
library(shiny)
library(shinychat)

ui <- bslib::page_fluid(
  
  chat_ui(
    "chat",
    messages = "**Привет!** Я ассистент по разработке кода на языке R. Чем могу помочь?",
    icon_assistant = htmltools::tags$img(
      src = "https://cdn.pixabay.com/photo/2017/03/31/23/11/robot-2192617_960_720.png",
      width = "40px",
      height = "40px"
    )
    )
)

server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
    )
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

Функция `chat_ui()` создаёт интерфейс чата, с помощью аргумента `messages` вы можете задавать приветственное сообщение, аргумент `icon_assistant` позволяет задавать собственную иконку AI ассистента обернув её в `htmltools::tags$img()`.

В серверной части вашего приложения с помощью пакета `ellmer` вам необходимо создать объект chat, и далее через `observeEvent` реагировать на отправку пользователем сообщений.

Показанный выше пример кода создаёт следующий интерфейс:

<img src="img/5-1.png" align="middle" width="640">

#### Добаление подсказок в чат
Пакет `shinychat` поддерживает добавление в чат подсказок, создать которые можно с помощью тега `<span>` с классом `suggestion` и `suggestion submit` для мгновенной отправки текста подсказки в чат.


``` r
library(shiny)
library(shinychat)

messages <-
  '
  **Привет!** Я ассистент по разработке кода на языке R. Чем могу помочь?

  Возможно вас интересует:

  * <span class="suggestion submit">Что такое язык R?</span>
  * <span class="suggestion">Напиши код на языке R, который </span>
  '

ui <- bslib::page_fluid(
  chat_ui(
    "chat",
    messages = messages
  )
)

server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
    )
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

<img src="img/5-2.png" align="middle" width="640">

Использование подсказок ассистентом также можно указать в системном промпте, например:

```
Если ты считаешь уместным предложить пользователю варианты ответов, которые он может захотеть написать, заключи текст каждого варианта в теги `<span class="suggestion">`.

Также используйте `"Предлагаемые следующие шаги:"` для представления предложений. Например:

1. <span class="suggestion">Вариант 1.</span>
2. <span class="suggestion">Вариант 2.</span>
3. <span class="suggestion">Вариант 3.</span>

```

#### Закрепить поле ввода запроса в нижней части экрана и настройка тем
В графических интерфейсах большинства LLM провайдеров окно ввода запроса закрепляется в нижней части окна, `shinychat` так же позволяет повторить это поведение. 

Для этого рендеринг страницы реализуйте с помощью функции `bslib::page_fillable()` и используйте её аргумент `fillable_mobile = TRUE`.
Все функции рендеринга страниц `page_*()` из пакета `bslib` имеют аргумент `theme`, который позволяет кастомировать тему вашей страницы. В данный аргумент необходимо передавать функцию `bslib::bs_theme()`, с помощью аргументов которой можно либо настроить тему самостоятельно, либо передать в аргумент `preset` название одной из преднастроенных тем, полный список преднастроенных тем можно получить с помощью функции `bslib::bootswatch_themes()`.


``` r
library(shiny)
library(shinychat)
library(bslib)

ui <- bslib::page_fillable(
  
  chat_ui(
    "chat",
    messages = "**Привет!** Я ассистент по разработке кода на язке R. Чем могу помочь?"
  ), 
  fillable_mobile = TRUE, 
  theme = bslib::bs_theme(preset = "darkly") # просмотрт доступных пресетов bslib::bootswatch_themes()
  
)

server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
  )
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

Данный интерфейс имеет тёмную тему, и окно ввода запроса всегда располагается в нижней части окна:

<img src="img/5-3.png" align="middle" width="640">

#### Распологаем интерфейс чата в sidebar
В плане интерфейса-помощника иногда удобно распологать чат в боковой панели (sidebar), для рендеринга в таком случае используйте `bslib::page_sidebar()`, а интерфейс чата (функцию `chat_ui()`) передавайте передавайте в аргумент `sidebar`, также предварительно завернув в функцию `sidebar()`.


``` r
library(shiny)
library(shinychat)
library(bslib)

ui <- bslib::page_sidebar(
  sidebar = sidebar(
    chat_ui(
      "chat",
      messages = list(
        "**Привет!** Я ассистент по разработке кода на язке R. Чем могу помочь? <span class='suggestion'>suggestion</span>."
      ),
      height = "100%"
    ),
    width = 500,
    style = "height: 100%;",
    position = 'right', 
    title = 'AI ассистент'
  ),
  "Main content",
  fillable = TRUE
)

server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
  )
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

<img src="img/5-4.png" align="middle" width="640">

#### Интерфейс чата внутри карточки
Иногда вам может понадобится отделить интерфейс чата от остальных элементов вашего приложения, для этого удобно поместить его в карточку с помощью функции `card()`.


``` r
library(shiny)
library(bslib)
library(shinychat)

ui <- page_fillable(
  card(
    card_header(
      "Welcome to Posit chat",
      tooltip(icon("question"), "This chat is brought to you by Posit."),
      class = "d-flex justify-content-between align-items-center"
    ),
    chat_ui(
      id = "chat",
      messages = "Hello! How can I help you today?"
    )
  ),
  fillable_mobile = TRUE
)


server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
  )
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

<img src="img/5-5.png" align="middle" width="640">

### Кастомизация вызова инструментов
В предыдущих уроках мы с вами разобрались с тем, как добавить в модель возможность вызова различных инструментов. Но, по умолчанию в интерфейсе чата вы не видите, когда и какие инструменты ассистенты использует, и тем более не видите какие аргументы в функции вызова инструментов передаются, и какой результат от вызова инструмента модель получила. А вся эта информация полезна как на этапе отладки вашего приложения, так и в целом пользователям вашего AI ассистента. 

Пакет `shinychat` позволяет добавлять в интерфейс чата информацию о вызываемых моделью инструментов. В этом примере мы напишем небольшую функцию для запроса прогноза погоды в которую необходимо передать координаты населёного пункта.

Код инструмента выглядит следующим образом:


``` r
library(weathR) # for forecasts via `point_tomorrow()`

get_weather_forecast <- tool(
  function(lat, lon) {
    point_tomorrow(lat, lon, short = FALSE)
  },
  name = "get_weather_forecast",
  description = "Get the weather forecast for a location.",
  arguments = list(
    lat = type_number("Latitude"),
    lon = type_number("Longitude")
  ),
  annotations = tool_annotations(
    title = "Запрос прогноза погоды",
    icon = bsicons::bs_icon("cloud-sun")
  )
)
```

Аргумент `annotations` принимает описание название и иконки вызыввемого инструмента, обёрнутых в функцию `tool_annotations()`.

Для того, что бы вызов инструмента отображлся в интерфейсе чата, в серверной части приложения в `chat$stream_async()` необходимо передать `stream = "content"`.


``` r
library(shinychat)
library(ellmer)
library(weathR) # for forecasts via `point_tomorrow()`

get_weather_forecast <- tool(
  function(lat, lon) {
    point_tomorrow(lat, lon, short = FALSE)
  },
  name = "get_weather_forecast",
  description = "Get the weather forecast for a location.",
  arguments = list(
    lat = type_number("Latitude"),
    lon = type_number("Longitude")
  ),
  annotations = tool_annotations(
    title = "Запрос прогноза погоды",
    icon = bsicons::bs_icon("cloud-sun")
  )
)

ui <- bslib::page_fluid(
  chat_ui(
    "chat",
    messages = "Я могу рассказать вам прогноз погоды на ближайшее время в любом регтоне."
  )
)

server <- function(input, output, session) {
  chat <- ellmer::chat_google_gemini(
    system_prompt = 'Ты ассситент который умеет искать прогноз погоды по заданному региону или локации. По заданной локации ты ищешь координаты, и далее определяешь прогноз погоды.'
  )
  
  # добавляем инструмент
  chat$register_tool(get_weather_forecast)
  
  observeEvent(input$chat_user_input, {
    stream <- chat$stream_async(input$chat_user_input, stream = "content") # для отображения инструмента необходимо включить stream = "content"
    chat_append("chat", stream)
  })
}

shinyApp(ui, server)
```

Теперь в чате будет отображаться вызов инструментов, аргументы, которые модель передала при вызове, и полученный результат:

<img src="img/5-6.png" align="middle" width="640">

По умолчанию иконкой вызываемого инструмента является гаечный ключ, а в качестве названия выводится название вызываемой моделью фукнции. Но при создании инструмента мы передали в аргумент `annotations` кастомную иконку, и подпись. Но, вы можете более гибко управлять как самой иконкой, так и описанием. В нашем случае в описание мы можем добавить передачу названия населённого пункта по которому запрашиваем погоду, а иконку менять в зависимости от того какая в населённом пункте погода, если тепло подставить солнце, если холодно то снег, а если идёт дождь то подставить тучи с дождём. Для реализации такого подхода внутри инструмента вам необходимо использовать конструктор `ContentToolResult()`.


``` r
get_weather_forecast <- tool(
  function(lat, lon, location_name, `_intent`) {
    forecast <- point_tomorrow(lat, lon, short = FALSE)
    
    icon <- if (any(forecast$temp > 70)) {
      bsicons::bs_icon("sun-fill")
    } else if (any(forecast$temp < 45)) {
      bsicons::bs_icon("snow")
    } else {
      bsicons::bs_icon("cloud-sun-fill")
    }
    
    ContentToolResult(
      forecast,
      extra = list(
        display = list(
          title = paste("Прогноз погоды для", location_name),
          icon = icon
        )
      )
    )
  },
  name = "get_weather_forecast",
  description = "Get the weather forecast for a location.",
  arguments = list(
    lat = type_number("Latitude"),
    lon = type_number("Longitude"),
    location_name = type_string("Name of the location for display to the user"),
    `_intent` = type_string(                                                        # дополнительная подсказка в интерфейсе
      "A short snippet used for display purposes to explain the call to the user."
    )
  ),
  annotations = tool_annotations(
    title = "Запрос прогноза погоды",
    icon = bsicons::bs_icon("cloud-sun")
  )
)
```

<img src="img/5-7.png" align="middle" width="640">

Теперь иконка вызываемого инструмента меняется в зависимости от текущей погоды, в описании указывается название населённого пункта, а в правой части описания отдельно указана причина, по которой модель решила вызвать этот инструмент. Добавление этого описания реализовано через специальный аргумент `_intent`, который вам надо добавить в функцию вызываемую инструментом, и в аннотации с описанием аргументов указать, что модель должна указывать в жтом аргументе причину вызова инструмента.

Динамически изменяющуюся иконку и описание инструмента мы реализовали с помощью конструктора `ContentToolResult()`, передав в аргумент `extra` описание через список `display` и его элементы `title` и `icon`. Список `display` также позволяет кастомизировать и вывод полученного инструментом результата, по умолчанию полученный результат выводится в виде блока кода, но используя элементы `html`, `markdown` и `text` списка `display` вы можете отформатировать вывод результат. 

Ниже пример вызываемого инструмента, в котором вывод прогноза погоды приводится к табличному виду:


``` r
get_weather_forecast <- tool(
  function(lat, lon, location_name) {
    forecast_data <- point_tomorrow(lat, lon, short = FALSE)
    forecast_table <- gt::as_raw_html(gt::gt(forecast_data)) # формируем HTML таблицу

    ContentToolResult(
      forecast_data,
      extra = list(
        display = list(
          html = forecast_table,                             # кастомизируем вывод инструмента
          title = paste("Weather Forecast for", location_name)
        )
      )
    )
  },
  name = "get_weather_forecast",
  description = "Get the weather forecast for a location.",
  arguments = list(
    lat = type_number("Latitude"),
    lon = type_number("Longitude"),
    location_name = type_string("Name of the location for display to the user")
  ),
  annotations = tool_annotations(
    title = "Weather Forecast",
    icon = bsicons::bs_icon("cloud-sun")
  )
)
```

Теперь вывод информации о погоде выглядит так:

<img src="img/5-8.png" align="middle" width="640">

Так же вы можете отформатировать вывод результата через Markdown:


``` r
get_weather_forecast <- tool(
  function(lat, lon, location_name) {
    forecast_data <- point_tomorrow(lat, lon, short = FALSE)

    temp_current <- forecast_data$temp[1]
    skies_current <- forecast_data$skies[[1]]

    temp_high <- max(forecast_data$temp)
    temp_low <- min(forecast_data$temp)

    humidity <- round(mean(forecast_data$humidity), 1)
    skies <- table(forecast_data$skies)
    skies <- names(skies)[which.max(skies)]

    forecast_summary <- glue::glue(
      "В **{location_name}** сейчас {temp_current}°F, _{tolower(skies_current)}_ небо. ",
      "Сегодня максимальная температура {temp_high}°F, минимальная — {temp_low}°F. ",
      "Влажность около {humidity}%. ",
      "В течение дня ожидается **{tolower(skies)}** небо."
    )

    ContentToolResult(
      forecast_data,
      extra = list(
        display = list(
          markdown = forecast_summary,
          title = paste("Weather Forecast for", location_name)
        )
      )
    )
  },
  name = "get_weather_forecast",
  description = "Get the weather forecast for a location.",
  arguments = list(
    lat = type_number("Latitude"),
    lon = type_number("Longitude"),
    location_name = type_string("Name of the location for display to the user")
  ),
  annotations = tool_annotations(
    title = "Weather Forecast",
    icon = bsicons::bs_icon("cloud-sun")
  )
)
```

<img src="img/5-9.png" align="middle" width="640">

Глобально управлять настройками отображения вызываемых инструментов можно с помощью опции `shinychat.tool_display` или переменной среды `SHINYCHAT_TOOL_DISPLAY`:

* `options(shinychat.tool_display = "none")`: отключить вывод информации о вызываемом инструменте
* `options(shinychat.tool_display = "basic")`: вывод базовой информации
* `options(shinychat.tool_display = "rich")`: вывод полный информации (значение по умолчанию)

### Настройка серверной части
#### Восстановление чата
По умолчанию если вы обновите вкладку браузера вся история вашего диалога с ассистентом сбросится, но с помощью функции `chat_restore()` это поведение можно исправить. Добавив её в серверную часть вашего приложения история вашей переписки будет фиксироваться в URL параметрах, после чего вы сможете по ссылке восстанавливать ваши диалоги, и при этом обновление вкладки так, же не сбросит историю:


``` r
library(shiny)
library(shinychat)

ui <- function(request) {
  bslib::page_fluid(
    chat_ui(
      "chat",
      messages = "**Привет!** Я ассистент по разработке кода на языке R. Чем могу помочь?"
    )
  )
}

server <- function(input, output, session) {
  chat_client <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
  )
  
  chat_restore(
    id = "chat",
    client = chat_client
  )

  observeEvent(input$chat_user_input, {
    stream <- chat_client$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server, enableBookmarking = "url")
```

#### Сброс чата
Реализовать функционал сброса чата можно добавив в серверную часть модуля вызов функции `chat_clear()`.


``` r
library(shiny)
library(shinychat)

ui <- function(request) {
  bslib::page_fluid(
    chat_ui(
      "chat",
      messages = "**Привет!** Я ассистент по разработке кода на языке R. Чем могу помочь?"
    ),
    actionButton("clear", "Clear chat")
  )
}

server <- function(input, output, session) {
  chat_client <- ellmer::chat_google_gemini(
    system_prompt = 'Ты помощник по разработке кода на языке R.'
  )
  
  observeEvent(input$clear, {
    chat_clear("chat")
  })
  
  observeEvent(input$chat_user_input, {
    stream <- chat_client$stream_async(input$chat_user_input)
    chat_append("chat", stream)
  })
}

shinyApp(ui, server, enableBookmarking = "url")
```

#### Использование shinychat с модульными Shiny приложениями
Всё, что мы рассматривали выше будет работать с простыми Shiny приложениями, но функции `chat_ui()` не работает с модулями, т.к. она не поддерживает автоматический namespace модулей. 

В сложных, больших приложениях реализованных через модульную систему Shiny вам необходимо использовать отдельные функции `chat_mod_ui()` + `chat_mod_server()`. Эти функции специально созданы для работы с модулями Shiny.


``` r
library(shiny)
library(shinychat)
library(ellmer)
library(bslib)

# ===== UI МОДУЛЯ =====
chatModuleUI <- function(id, greeting = "Привет! Чем могу помочь?") {
  chat_mod_ui(
    id, 
    messages = greeting
  )
}

# ===== SERVER МОДУЛЯ =====
chatModuleServer <- function(id, system_prompt = "Ты помощник.") {
  # Создаём клиент
  chat_client <- ellmer::chat_google_gemini(
    system_prompt = system_prompt
  )
  
  # Запускаем chat_mod_server (он сам создаёт moduleServer внутри)
  chat_mod_server(id, chat_client)
}

# ===== ГЛАВНОЕ ПРИЛОЖЕНИЕ =====
ui <- page_fillable(
  titlePanel("Пример чата с модулями Shiny"),
  
  layout_columns(
    col_widths = c(6, 6),
    
    # Первый чат
    card(
      card_header("Чат 1 - Помощник по R"),
      chatModuleUI(
        id = "chat1",
        greeting = "Привет! Я помогу с вопросами по R."
      )
    ),
    
    # Второй чат
    card(
      card_header("Чат 2 - Помощник по Python"),
      chatModuleUI(
        id = "chat2",
        greeting = "Привет! Я помогу с вопросами по Python."
      )
    )
  )
)

server <- function(input, output, session) {
  # Инициализируем модули
  chatModuleServer(
    id = "chat1",
    system_prompt = "Ты эксперт по языку программирования R."
  )
  
  chatModuleServer(
    id = "chat2",
    system_prompt = "Ты эксперт по языку программирования Python."
  )
}

shinyApp(ui, server)
```

Таким образом мы реализовали модульную логику, создали модуль создания чата, и в основном приложении дважды вызвали его, создав два парллельных чата, один как ассистент по разработке на языке R, второй по python.

<img src="img/5-10.png" align="middle" width="640">

Функция `chat_mod_ui()`:

1. Автоматически создаёт `ns <- NS(id)`
2. Применяет `namespace` ко ВСЕМ внутренним элементам
3. Возвращает готовый UI с правильными ID

Функция `chat_mod_server()`:

1. Автоматически вызывает `moduleServer(id, function(...) {...})`
2. Настраивает `observeEvent` для обработки сообщений
3. Обрабатывает стриминг ответов от LLM
4. Возвращает объект с методами для управления чатом

## Заключение
Мы рассмотрели возможности пакета `shinychat`, которые позволяют превратить простой чат в полноценное бизнес-приложение. Теперь вы знаете, как кастомизировать внешний вид ассистента, использовать систему подсказок, делать работу инструментов прозрачной для пользователя и масштабировать чат в рамках сложных модульных систем Shiny.

Грамотно настроенный интерфейс не только радует глаз, но и значительно снижает порог входа для пользователей вашего AI-инструмента.

## Вопросы для самопроверки

1. <details>
    <summary>**В чем принципиальная разница между классами `suggestion` и `suggestion submit` в подсказках?**</summary>
    Класс `suggestion` просто подставляет текст подсказки в поле ввода, позволяя пользователю его дополнить или отредактировать. Класс `suggestion submit` не только подставляет текст, но и мгновенно отправляет его в чат как готовый запрос.
   </details>

2. <details>
    <summary>**Зачем в методе `stream_async()` указывать аргумент `stream = "content"`, если мы работаем с инструментами?**</summary>
    По умолчанию `shinychat` скрывает технические подробности вызова инструментов. Установка `stream = "content"` активирует отображение специальных карточек в чате, которые показывают пользователю (или разработчику при отладке), какой инструмент вызван, с какими аргументами и какой результат получен.
   </details>

3. <details>
    <summary>**Как аргумент `_intent` внутри функции-инструмента помогает улучшить UI чата?**</summary>
    Если добавить этот аргумент в функцию и описать его в `arguments`, модель сама сгенерирует краткое пояснение (намерение), зачем она вызывает этот инструмент. Это пояснение отобразится в заголовке карточки инструмента в интерфейсе, делая работу AI более прозрачной для пользователя.
   </details>

4. <details>
    <summary>**Какую роль играет функция `chat_restore()` и почему для её работы нужен `enableBookmarking = "url"`?**</summary>
    `chat_restore()` извлекает историю переписки из URL-параметров при загрузке страницы. Чтобы Shiny мог сохранять и считывать эти данные в адресной строке, в функции `shinyApp` обязательно должен быть включен механизм букмаркинга.
   </details>

5. <details>
    <summary>**Почему в больших проектах нельзя просто использовать `chat_ui()` внутри модулей Shiny?**</summary>
    Обычная `chat_ui()` не умеет автоматически работать с пространствами имен (`ns()`). Из-за этого ID элементов внутри модуля могут конфликтовать или не считываться сервером. Специальные функции `chat_mod_ui()` и `chat_mod_server()` созданы специально для бесшовной интеграции чата в модульную структуру Shiny.
   </details>
   
