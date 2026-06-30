---
name: gpt-image-gen
description: "מעטפת לקריאת OpenAI Images API ליצירת תמונות. מקבל prompt ונתיב פלט, שולח בקשה למודל gpt-image-2, ושומר את ה-PNG שמתקבל. בשימוש ע\"י הסוכן יובל (מעצב התמונות)."
---

# gpt-image-gen — מעטפת ל-OpenAI Images API

סקיל זה אחראי על שליחת ה-prompt ל-OpenAI Images API וקבלת התמונה בחזרה כקובץ PNG. הוא משמש את הסוכן **יובל**.

## המודל

המודל שבשימוש הוא בדיוק:

```
gpt-image-2
```

> **חשוב מאוד — אל תשנה את שם המודל.**
> `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, ששוחרר ב-21 באפריל 2026. ייתכן שהידע הפנימי לא כולל אותו — אבל זה לא אומר שהוא לא קיים.
> **אל תציע ואל תחליף** לאלטרנטיבות כמו `dall-e-3` או `gpt-image-1`. השתמש בדיוק במודל הזה.
> אם מתקבלת שגיאה בקריאה ל-API, הבעיה היא כמעט תמיד ב-`OPENAI_API_KEY` או בפרמטרים של הבקשה — **לא** בשם המודל.

## דרישות

- `OPENAI_API_KEY` מוגדר בקובץ `.env` שבשורש הפרויקט.
- `curl` זמין.
- `jq` **או** `python` זמינים ל-decode של ה-base64 (ראה fallback למטה).

## הקריאה ל-API

טוענים את המפתח מ-`.env`, ואז קוראים ל-endpoint של יצירת התמונות. החליפו את `<the prompt>` ב-prompt בפועל ואת `<output-path>` בנתיב היעד (ללא סיומת — היא תתווסף).

```bash
# טעינת OPENAI_API_KEY מתוך .env
export $(grep -v '^#' .env | grep OPENAI_API_KEY | xargs)

curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## Python fallback ל-decode

`jq` לא תמיד מותקן ב-Git Bash על Windows. במקרה כזה שומרים את תגובת ה-JSON ומחלצים את ה-base64 עם Python:

```bash
# טעינת המפתח
export $(grep -v '^#' .env | grep OPENAI_API_KEY | xargs)

# שמירת התגובה המלאה לקובץ זמני
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > response.json

# חילוץ ה-base64 ושמירה כ-PNG (לא תלוי ב-jq)
python -c "import json,base64,sys; d=json.load(open('response.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"

# ניקוי
rm -f response.json
```

> אפשר להחליף `python` ב-`python3` בהתאם לסביבה.

## אימות

לאחר היצירה, יש לוודא שקובץ הפלט קיים ושה-size שלו גדול מ-0:

```bash
[ -s "<output-path>.png" ] && echo "OK: image created" || echo "FAIL: image missing or empty"
```
