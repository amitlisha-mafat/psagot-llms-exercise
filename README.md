# Basic LLM Agents Exercise: scaffold

תרגיל בן ~3.5 שעות: להפוך את ה-RAG של הבוקר ל-tool אחד בתוך agent שנכתב ביד.

**המחברת:** [basic_agents_exercise.ipynb](basic_agents_exercise.ipynb)

ההוראות במחברת בעברית, הכותרות ושמות המזהים באנגלית. המבנה הפדגוגי, שלד לולאת
ה-agent, ה-dispatch, ה-evaluation וניתוח הכשלים, מוכנים ולא תלויי דומיין. חמישה
דברים ריקים בכוונה.

## 🔴 למלא לפני העברת התרגיל

לחפש במחברת `PLACEHOLDER` (21 סימונים) ו-`<<REPLACE` (39 סימונים).

| # | Placeholder | סעיף | מה צריך |
|---|---|---|---|
| 1 | Scenario | §1 | התרחיש, בהמשך לתרגיל הבוקר, וטבלת חמש שאלות הדוגמה |
| 2 | Corpus | §3.1 | 10–15 מסמכים במבנה `doc_id` / `title` / `date` / `text` |
| 3 | Provider adapter | §2.2 | `to_provider_tools`, `call_llm`, `make_tool_result_messages` |
| 4 | Capabilities | §4.1 | ה-API / client / פונקציות שהסטודנטים מקבלים ועוטפים |
| 5 | Dev set | §10.1 | 10–15 שאלות, שבע קטגוריות. שורה אחת כתובה כתבנית |

בנוסף, קטנים יותר: `SYSTEM_PROMPT` (§8), `PROBE_QUESTIONS` (§5),
`SCRATCH_QUESTION` (§6), ושאלות ההדגמה ב-§9.

### אילוץ תוכן

מה שתיתנו ב-§4 חייב להחזיק עובדות שאין באף מסמך, ולהפך. אם מקור אחד עונה על הכול,
ה-agent אף פעם לא צריך **לבחור**, והתרגיל מתמוטט ל-RAG עם צעדים מיותרים. רצוי שמסמך
אחד יפנה למקור השני כסמכותי לשדות האלה. אז agent שעונה עליהם מהארכיון נראה שגוי
ב-trace.

לשתול **שרשרת תלות** אחת לפחות: מסמך מזכיר ישות לפי מזהה ← ה-capability מתרגם אותו
← קריאה נוספת משלימה. זו השרשרת שהופכת את §9 לאמיתי, וזה מה שאי אפשר לקבע מראש.

## התקנה

```sh
pip install -r requirements.txt          # opik + jupyterlab
pip install <your-model-provider-sdk>
pip install <any client library your §4 capabilities need>

export OPIK_API_KEY=...        # Opik Cloud
export OPIK_WORKSPACE=...
# או self-hosted:  export OPIK_USE_LOCAL=true

jupyter lab
```

אם Opik לא מוגדר, ה-tracing הופך ל-no-op והמחברת עדיין רצה, אבל כל התרגיל בנוי על
קריאת traces, אז כדאי לתקן לפני שמתחילים.

## RTL

תאי ה-markdown עטופים ב-`<div dir="rtl">`, ובלוקי קוד ודיאגרמות ב-`<div dir="ltr">`
כדי שלא יתהפכו. הטריק הוא שורה ריקה אחרי תגית ה-div הפותחת: בלוק HTML מסתיים בשורה
ריקה, ולכן מה שאחריה עדיין נפרס כ-markdown רגיל.

מבוסס על תכונת `dir`, לא על CSS, כדי שישרוד גם ב-VS Code וב-GitHub שמסננים `<style>`.
תגית `<style>` בתא הראשון מוסיפה יישור וכיווניות ל-`code` ולטבלאות היכן שהיא כן עוברת.

**בעריכה:** לשמור על העטיפה. תא markdown חדש צריך

```
<div dir="rtl">

<תוכן>

</div>
```

עם שורות ריקות משני צידי התוכן. בלוק קוד או דיאגרמת ASCII: `dir="ltr"` בנפרד, לא
בתוך בלוק ה-rtl.

## §4: היכולות שאתם נותנים

§4 דק בכוונה. הוא מניח שהמקור השני **ניתן**, לא נבנה: client פנימי שהקורס כבר משתמש
בו, endpoint, handle למאגר, פונקציה שמישהו אחר כתב. הסטודנטים לא מממשים אותו, הם
**עוטפים אותו כ-tool**, וזו המיומנות ששווה ללמד:

| נבנה למתכנת | מה שהמודל צריך |
|---|---|
| אובייקט מקונן / JSON גולמי | טקסט שטוח, קריאה אחת |
| זורק חריגה על מזהה שגוי | משפט שאומר שהמזהה לא נמצא |
| 12 פרמטרים, 3 חובה | מעט ארגומנטים, כולם אופציונליים |
| מחזיר 400 שורות | הבודדות שעונות על השאלה |
| שדה בשם `st_cd` | משהו שהתיאור יכול להסביר |

§4.1 צריך רק להשאיר שתיים-שלוש callables ב-scope. שני stubs מסופקים מאחורי
`USE_STUB_CAPABILITIES` כדי שהמחברת תרוץ לפני חיבור אמיתי.

**לתעד באותו תא מה היכולת מחזירה ואיך היא נכשלת.** ה-stubs מחזירים `list[dict]`
ו-`[]` על miss; אם ה-API האמיתי זורק על miss או מחזיר מעטפת מקוננת, לכתוב את זה,
כי העטיפה ב-§4.2 חייבת להתמודד והסטודנטים לא יכולים לנחש.

עדיף credentials של קריאה בלבד, מהסביבה. כל קריאה כאן היא קריאה, והארגומנטים
נקבעים על ידי ה-agent.

## אי-תלות בספק

הלולאה לא צריכה לדעת מי הספק. שלוש פונקציות מכירות אותו.

```text
         provider SDK
              |
   +----------+-----------+
   |  3 adapter functions |   <- the only provider-specific code (2.2)
   +----------+-----------+
              |
   LLMResponse / ToolCall     <- provider-neutral from here down
              |
   tools, dispatch, agent loop, evaluation
```

`LLMResponse.assistant_message` הוא מה שחשוב: תור ה-assistant בפורמט הספק, מוחזר
**כמו שהוא**. שם יושבים tool-call ids ו-thinking blocks שהבקשה הבאה צריכה. בנייה
מחדש מ-`.text` היא שגיאת API מיידית בחלק מהמודלים ודגרדציה שקטה באחרים. הבאג הנפוץ
ביותר בתרגיל, ומסומן ב-§2.2, §6 ו-§8.

`make_tool_result_messages` מחזירה **רשימה**, כדי שספק שרוצה הודעה אחת לכל תוצאה
וספק שרוצה הודעה אחת לכולן יתאימו שניהם בלי לשנות את הלולאה.

ה-schemas נכתבים בצורה ניטרלית (`name` / `description` / `parameters`);
`to_provider_tools` ממיר. הסטודנטים לא רואים פורמט wire.

## מבנה

| # | Section | מסופק | סטודנט |
|---|---|:--:|:--:|
| 0 | Fill in before teaching | ✓ | |
| 1 | Mission | 🔴 | קורא |
| 2 | Setup, tracing, provider contract | ✓ / 🔴 adapter | בוחן traces |
| 3 | RAG as a tool | corpus 🔴, retriever ✓ | כותב `search_documents` |
| 4 | Add more sources | 🔴 | כותב שתי עטיפות |
| 5 | Tool schemas | 1 מתוך 3 כתוב, probe ✓ | כותב 2, משווה traces |
| 6 | One manual cycle | שלד | ממלא 5 שלבים |
| 7 | Tool dispatch | שלד + בדיקות | כותב `execute_tool` |
| 8 | The agent loop | שלד, prompt 🔴 | כותב `run_agent` |
| 9 | Single-tool → multi-step | 🔴 שאלות | מריץ, קורא traces |
| 10 | Evaluation | grader + runner ✓, dev set 🔴 | מריץ |
| 11 | Optimization challenge | compare ✓ | מכוונן |
| 12 | Failure analysis | dump + תבנית ✓ | ממלא |
| 13 | Final reflection | ✓ | עונה |

58 תאים, 30 קוד, 8 תאי TODO.

## הערות למרצה

**לכתוב החטאות מכוונות ב-dev set.** דוגמה שהשאילתה המתבקשת מחזירה בה רק חצי ממה
שצריך, או ששני tools נשמעים בה סבירים, היא מה שהופך את §12 מטקס לאבחון אמיתי.
שתיים-שלוש בכוונה, ולסמן בעותק שלכם אילו הן.

**עלות.** ריצת `evaluate()` אחת = ריצת agent לכל דוגמה (כמה קריאות LLM כל אחת) ועוד
קריאת judge לכל דוגמה עם `judge`. הסטודנטים מריצים שוב ושוב ב-§11. לרכז את פרמטרי
העלות ב-`LLM_SETTINGS` כדי שיהיה מקום אחד להנמיך בו.

**מחוץ לתחום היום.** בלי planning, scratchpad, task decomposer או framework. השאלה
האחרונה ב-§13 מובילה ישירות לתרגיל המתקדם של יום ב'.

## מה נבדק

הורצו בנפרד: ה-retriever (כולל שאילתות בעברית), ה-stubs של §4, וחוזה
`LLMResponse` / `ToolCall` / `ToolResult`. כל 30 תאי הקוד עוברים parse. כל הכותרות באנגלית.
העטיפה ל-RTL אומתה מול renderer של CommonMark: כותרות, רשימות, טבלאות ובלוקי קוד
עדיין נפרסים כ-markdown בתוך ה-divs, תגיות ה-`dir` נשמרות, וכל ה-divs מאוזנים.

ה-tokenizer של ה-retriever מכסה `[a-z0-9֐-׿]`: לטינית, ספרות ועברית.
לקורפוס בשפה אחרת יש להרחיב את הביטוי.

לא נבדק: אין ריצה מקצה לקצה. היא דורשת adapter ויכולות אמיתיות, ושניהם placeholders
בכוונה.

## פתרונות

לא כלולים. להעתיק ל-`basic_agents_exercise_solutions.ipynb` ולמלא שם את ה-TODOs, כדי
שגרסת הסטודנטים תישאר נקייה.
