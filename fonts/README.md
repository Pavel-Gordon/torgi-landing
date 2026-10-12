# Шрифты лендинга

`Oswald-latin.woff2` и `Oswald-cyrillic.woff2` — сабсеты Oswald
(Google Fonts, переменное начертание 200–700, SIL OFL 1.1).

## Зачем они здесь, если не отдаются браузеру

Файлы **не подключаются как ссылки**. Они вшиты в `../styles.css` как
`data:font/woff2;base64,…` — потому что `payment.py` отдаёт ровно четыре
маршрута: `/`, `/styles.css`, `/download/<файл>` и `/pay`. Маршрута
`/fonts/<имя>.woff2` в нём нет, и ссылка на файл даёт 404: Oswald в
системе не установлен, и браузер молча уходит на системный шрифт —
заголовки выглядят как обычный sans-serif, а лендинг обещает Oswald.

Держать файлы здесь нужно, чтобы `data:URI` можно было пересобрать:

```sh
cd landing
python3 - <<'PY'
import base64, re
meta = [("latin",
         "U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+2000-206F, U+2122, "
         "U+2191, U+2193, U+2212, U+2215"),
        ("cyrillic",
         "U+0301, U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116")]
faces = []
for sub, urange in meta:
    b64 = base64.b64encode(open("fonts/Oswald-%s.woff2" % sub, "rb").read()).decode()
    faces.append('@font-face {\n'
                 '  font-family: "Oswald";\n'
                 '  font-style: normal;\n'
                 '  font-weight: 200 700;\n'
                 '  font-display: swap;\n'
                 '  src: url("data:font/woff2;base64,%s") format("woff2");\n'
                 '  unicode-range: %s;\n'
                 '}' % (b64, urange))
css = open("styles.css", encoding="utf-8").read()
css = css[:css.index("@font-face {")] + "\n".join(faces) + "\n\n" + \
      css[css.index(":root {"):]
open("styles.css", "w", encoding="utf-8").write(css)
PY
```

Диапазоны `unicode-range` совпадают с `app.py: THEME_FONT_META` — чтобы
сайт и приложение выглядели одним шрифтом.

## Откуда файлы

Скачаны с Google Fonts (latin и cyrillic отдельными подмножествами).
**Не копия `../../fonts/`** — там два из четырёх файлов повреждены, см.
`../../fonts/README.md` и задачу в `tasks.md`.