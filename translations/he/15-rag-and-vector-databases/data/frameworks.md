<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "b5466bcedc3c75aa35476270362f626a",
  "translation_date": "2025-07-09T16:34:09+00:00",
  "source_file": "15-rag-and-vector-databases/data/frameworks.md",
  "language_code": "he"
}
-->
# מסגרות רשתות עצביות

כפי שלמדנו כבר, כדי לאמן רשתות עצביות בצורה יעילה אנחנו צריכים לעשות שני דברים:

* לפעול על טנסורים, למשל לכפול, לחבר ולחשב פונקציות כמו סיגמואיד או סופטמקס  
* לחשב נגזרות של כל הביטויים, כדי לבצע אופטימיזציית ירידת גרדיאנט

בעוד שספריית `numpy` יכולה לבצע את החלק הראשון, אנחנו צריכים מנגנון כלשהו לחישוב נגזרות. במסגרת שפיתחנו בסעיף הקודם היינו צריכים לתכנת ידנית את כל פונקציות הנגזרת בתוך המתודה `backward`, שמבצעת את ה-backpropagation. באופן אידיאלי, מסגרת עבודה צריכה לאפשר לנו לחשב נגזרות של *כל ביטוי* שנוכל להגדיר.

דבר חשוב נוסף הוא היכולת לבצע חישובים על GPU, או יחידות חישוב מיוחדות אחרות, כמו TPU. אימון רשתות עצביות עמוקות דורש *הרבה* חישובים, ולכן היכולת לפרלל את החישובים האלה על GPUs היא מאוד חשובה.

> ✅ המונח 'לפרלל' משמעותו לפזר את החישובים על פני מספר מכשירים.

כיום, שתי המסגרות הפופולריות ביותר הן: TensorFlow ו-PyTorch. שתיהן מספקות API ברמת נמוכה לעבודה עם טנסורים הן על CPU והן על GPU. מעל ה-API ברמת הנמוכה, קיימות גם API ברמת גבוהה, הנקראות בהתאמה Keras ו-PyTorch Lightning.

| Low-Level API | TensorFlow | PyTorch |
|--------------|------------|---------|
| High-level API | Keras | Pytorch |

**API ברמת נמוכה** בשתי המסגרות מאפשרים לבנות מה שנקרא **גרפים חישוביים**. גרף זה מגדיר כיצד לחשב את הפלט (בדרך כלל פונקציית האובדן) עם פרמטרי קלט נתונים, וניתן לדחוף אותו לחישוב על GPU אם זמין. קיימות פונקציות לגזירת הגרף החישובי ולחישוב נגזרות, שניתן להשתמש בהן לאופטימיזציה של פרמטרי המודל.

**API ברמת גבוהה** מתייחסים לרשתות עצביות בעיקר כאל **רצף של שכבות**, ומקלים מאוד על בניית רוב הרשתות העצביות. אימון המודל בדרך כלל דורש הכנת הנתונים ואז קריאה לפונקציית `fit` שתבצע את העבודה.

ה-API ברמת גבוהה מאפשר לבנות רשתות עצביות טיפוסיות במהירות מבלי לדאוג להרבה פרטים. במקביל, ה-API ברמת נמוכה מציע שליטה רבה יותר על תהליך האימון, ולכן הוא בשימוש נרחב במחקר, כאשר עובדים עם ארכיטקטורות חדשות של רשתות עצביות.

חשוב גם להבין שניתן להשתמש בשתי ה-API יחד, למשל לפתח ארכיטקטורת שכבה ברמת נמוכה ואז להשתמש בה בתוך רשת גדולה יותר שנבנתה ואומנה עם ה-API ברמת גבוהה. או להגדיר רשת באמצעות ה-API ברמת גבוהה כרצף שכבות, ואז להשתמש בלולאת אימון ברמת נמוכה משלך לביצוע האופטימיזציה. שתי ה-API מבוססות על אותם מושגים בסיסיים, והן מתוכננות לעבוד היטב יחד.

## למידה

בקורס זה, אנו מציעים את רוב התוכן הן עבור PyTorch והן עבור TensorFlow. ניתן לבחור את המסגרת המועדפת ולעבור רק על המחברות המתאימות. אם אינכם בטוחים איזו מסגרת לבחור, מומלץ לקרוא דיונים באינטרנט בנושא **PyTorch מול TensorFlow**. אפשר גם להסתכל על שתי המסגרות כדי לקבל הבנה טובה יותר.

כאשר אפשר, נשתמש ב-High-Level APIs לפשטות. עם זאת, אנו סבורים שחשוב להבין כיצד רשתות עצביות פועלות מהיסוד, ולכן בתחילה נתחיל לעבוד עם ה-API ברמת נמוכה ועם טנסורים. אם אתם רוצים להתחיל מהר ואינכם רוצים להשקיע זמן רב בלמידת הפרטים האלה, ניתן לדלג עליהם ולעבור ישירות למחברות ברמת גבוהה.

## ✍️ תרגילים: מסגרות

המשיכו ללמוד במחברות הבאות:

| Low-Level API | TensorFlow+Keras Notebook | PyTorch |
|--------------|----------------------------|---------|
| High-level API | Keras | *PyTorch Lightning* |

לאחר שתשלוטו במסגרות, נחזור על מושג ה-overfitting.

# Overfitting

Overfitting הוא מושג חשוב מאוד בלמידת מכונה, וחשוב מאוד להבין אותו נכון!

נבחן את הבעיה של התאמת 5 נקודות (מיוצגות על ידי `x` בגרפים למטה):

| !linear | overfit |
|-------------------------|--------------------------|
| **מודל ליניארי, 2 פרמטרים** | **מודל לא ליניארי, 7 פרמטרים** |
| שגיאת אימון = 5.3 | שגיאת אימון = 0 |
| שגיאת ולידציה = 5.1 | שגיאת ולידציה = 20 |

* משמאל, רואים קירוב קו ישר טוב. מכיוון שמספר הפרמטרים מתאים, המודל תופס נכון את התבנית מאחורי התפלגות הנקודות.  
* מימין, המודל חזק מדי. מכיוון שיש רק 5 נקודות והמודל כולל 7 פרמטרים, הוא יכול להתאים את עצמו כך שיעבור דרך כל הנקודות, מה שמביא לשגיאת אימון של 0. עם זאת, זה מונע מהמודל להבין את התבנית האמיתית מאחורי הנתונים, ולכן שגיאת הוולידציה גבוהה מאוד.

חשוב מאוד למצוא איזון נכון בין העושר של המודל (מספר הפרמטרים) לבין מספר דגימות האימון.

## מדוע מתרחש overfitting

  * אין מספיק נתוני אימון  
  * מודל חזק מדי  
  * רעש רב מדי בנתוני הקלט

## כיצד לזהות overfitting

כפי שניתן לראות בגרף למעלה, ניתן לזהות overfitting על ידי שגיאת אימון נמוכה מאוד ושגיאת ולידציה גבוהה. בדרך כלל במהלך האימון נראה ששגיאות האימון והוולידציה יורדות, ואז בנקודה מסוימת שגיאת הוולידציה עלולה להפסיק לרדת ולהתחיל לעלות. זהו סימן ל-overfitting, ומדד לכך שכדאי להפסיק את האימון בנקודה זו (או לפחות לשמור snapshot של המודל).

overfitting

## כיצד למנוע overfitting

אם אתם רואים ש-overfitting מתרחש, ניתן לעשות אחד מהדברים הבאים:

 * להגדיל את כמות נתוני האימון  
 * להקטין את מורכבות המודל  
 * להשתמש בטכניקת רגולריזציה כלשהי, כמו Dropout, שנבחן בהמשך.

## Overfitting ו-Bias-Variance Tradeoff

Overfitting הוא למעשה מקרה של בעיה כללית יותר בסטטיסטיקה הנקראת Bias-Variance Tradeoff. אם נבחן את מקורות השגיאה האפשריים במודל שלנו, נוכל לראות שני סוגי שגיאות:

* **שגיאות הטיה (Bias)** נגרמות מכך שהאלגוריתם שלנו לא מצליח ללכוד נכון את הקשר בין נתוני האימון. זה יכול לנבוע מכך שהמודל שלנו לא חזק מספיק (**underfitting**).  
* **שגיאות שונות (Variance)** נגרמות מכך שהמודל מתאים את עצמו לרעש בנתוני הקלט במקום לקשר משמעותי (**overfitting**).

במהלך האימון, שגיאת ההטיה יורדת (כשהמודל לומד להתאים את הנתונים), ושגיאת השונות עולה. חשוב להפסיק את האימון - או ידנית (כשמזהים overfitting) או אוטומטית (על ידי הכנסת רגולריזציה) - כדי למנוע overfitting.

## סיכום

בשיעור זה למדתם על ההבדלים בין ה-API השונים בשתי המסגרות הפופולריות ביותר, TensorFlow ו-PyTorch. בנוסף, למדתם על נושא חשוב מאוד, ה-overfitting.

## 🚀 אתגר

במחברות הנלוות תמצאו 'משימות' בתחתית; עברו על המחברות והשלימו את המשימות.

## סקירה ולמידה עצמית

עשו מחקר על הנושאים הבאים:

- TensorFlow  
- PyTorch  
- Overfitting  

שאלו את עצמכם את השאלות הבאות:

- מה ההבדל בין TensorFlow ל-PyTorch?  
- מה ההבדל בין overfitting ל-underfitting?

## מטלה

במעבדה זו, עליכם לפתור שתי בעיות סיווג באמצעות רשתות fully-connected חד-שכבתיות ורב-שכבתיות, תוך שימוש ב-PyTorch או TensorFlow.

**כתב ויתור**:  
מסמך זה תורגם באמצעות שירות תרגום מבוסס בינה מלאכותית [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון כי תרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. המסמך המקורי בשפת המקור שלו נחשב למקור הסמכותי. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אנושי. אנו לא נושאים באחריות לכל אי-הבנה או פרשנות שגויה הנובעת משימוש בתרגום זה.