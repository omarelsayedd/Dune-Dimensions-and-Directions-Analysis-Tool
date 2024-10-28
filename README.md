### توثيق أداة تحليل أبعاد واتجاهات الكثبان الرملية

#### اسم الأداة
**تحليل أبعاد واتجاهات الكثبان الرملية (Dune Dimensions and Directions Analysis Tool)**

#### وصف الأداة
تقوم هذه الأداة بتحليل خصائص هندسية وجغرافية للكثبان الرملية المخزنة كطبقة بوليغون. حيث تقوم بحساب مساحة الكثيب، محيطه، طوله، عرضه، واتجاهات الطول والعرض بالدرجات والنصوص ضمن الاتجاهات الثمانية الأساسية. وتساعد هذه الأداة في توفير معلومات دقيقة عن الكثبان الرملية التي يمكن استخدامها في دراسات الجيومورفولوجيا وتحليل التضاريس.

---

### المدخلات
1. **طبقة البوليغون (polygon_layer)**: 
   - مسار طبقة الكثبان الرملية المخزنة في قاعدة البيانات الجغرافية (Geodatabase) بصيغة بوليغون.
   - مثال: `r"C:\Users\omare\OneDrive\Desktop\Dune Project\GeoDatabase\Dune_Vector.gdb\dunes_2008"`

---

### الحقول الجديدة
الأداة ستتحقق من وجود الحقول التالية في جدول السمات الخاص بالطبقة. إذا لم تكن موجودة، سيتم إضافتها تلقائيًا:

| الحقل                     | النوع      | الوصف |
|---------------------------|------------|-------|
| `Area`                    | DOUBLE     | مساحة الكثيب بالمتر المربع. |
| `Perimeter`               | DOUBLE     | محيط الكثيب بالمتر. |
| `Length`                  | DOUBLE     | الطول (أكبر بُعد) للكثيب بالمتر. |
| `Length_Direction_Degree` | DOUBLE     | زاوية اتجاه الطول بالدرجة. |
| `Length_Direction_Text`   | TEXT       | اتجاه الطول كنص (الاتجاهات الثمانية). |
| `Width`                   | DOUBLE     | العرض (أصغر بُعد) للكثيب بالمتر. |
| `Width_Direction_Degree`  | DOUBLE     | زاوية اتجاه العرض بالدرجة. |
| `Width_Direction_Text`    | TEXT       | اتجاه العرض كنص (الاتجاهات الثمانية). |

---

### خطوات الكود
1. **استيراد المكتبات**:
   - `arcpy`: مكتبة ArcGIS للعمل مع البيانات الجغرافية.
   - `numpy`: مكتبة للحسابات الرياضية، تستخدم هنا لحساب الزوايا.

2. **تحديد طبقة البوليغون**:
   - تم تعيين مسار طبقة البوليغون الخاصة بالكثبان الرملية.

3. **دالة تحويل الزاوية إلى اسم الاتجاه**:
   - دالة `angle_to_direction` تأخذ زاوية معينة بالدرجة وتحولها إلى نص الاتجاه الأقرب ضمن الاتجاهات الثمانية (شمال، شمال شرق، شرق، جنوب شرق، جنوب، جنوب غرب، غرب، شمال غرب).

4. **التحقق من وجود الحقول وإضافتها إذا كانت غير موجودة**:
   - الكود يتحقق من وجود الحقول المطلوبة (`Area`, `Perimeter`, `Length`, إلخ) في جدول السمات الخاص بالطبقة. إذا لم يكن الحقل موجودًا، يتم إنشاؤه تلقائيًا باستخدام `AddField_management`.

5. **استخدام `UpdateCursor` لتحديث القيم في الحقول الجديدة**:
   - يقوم `UpdateCursor` بتصفح كل كثيب في طبقة البوليغون وتحديث القيم المحسوبة.

6. **حساب المساحة، المحيط، الطول، والعرض**:
   - `SHAPE@AREA` و `SHAPE@LENGTH` هما خصائص متاحة ضمن الأداة لتوفير مساحة ومحيط البوليغون مباشرة.
   - `extent` للبوليغون يستخدم لحساب العرض والطول من خلال أخذ أبعاد المستطيل المحيط به.

7. **حساب اتجاه الطول والعرض**:
   - يتم حساب اتجاه الطول من خلال تحديد نقطتين على طول أكبر بُعد للبوليغون واستخدام `arctan2` لحساب الزاوية.
   - زاوية الطول تُحول إلى اتجاه نصي باستخدام الدالة `angle_to_direction`.
   - زاوية العرض يتم حسابها كزاوية عمودية على زاوية الطول (إضافة 90 درجة)، ثم تحويلها إلى اتجاه نصي.

8. **تحديث القيم في الجدول**:
   - بعد حساب القيم، يتم تحديث الحقول الجديدة في جدول السمات الخاص بالكثيب باستخدام `updateRow`.

9. **طباعة رسالة إتمام العملية**:
   - عند انتهاء العملية بنجاح، يتم طباعة رسالة "تمت إضافة الحقول (عند الحاجة) وتحديث البيانات بنجاح."

---

### الكود الكامل للأداة

```python
import arcpy
import numpy as np

# تحديد طبقة البوليجون
polygon_layer = r"C:\Users\omare\OneDrive\Desktop\Dune Project\GeoDatabase\Dune_Vector.gdb\dunes_2008"

# دالة لتحويل الزاوية إلى اسم اتجاه ضمن الاتجاهات الثمانية
def angle_to_direction(angle):
    directions = ["شمال", "شمال شرق", "شرق", "جنوب شرق", "جنوب", "جنوب غرب", "غرب", "شمال غرب"]
    idx = int(((angle + 22.5) % 360) / 45)
    return directions[idx]

# قائمة الحقول المراد إضافتها مع نوع البيانات
fields = [
    ("Area", "DOUBLE"),
    ("Perimeter", "DOUBLE"),
    ("Length", "DOUBLE"),
    ("Length_Direction_Degree", "DOUBLE"),
    ("Length_Direction_Text", "TEXT"),
    ("Width", "DOUBLE"),
    ("Width_Direction_Degree", "DOUBLE"),
    ("Width_Direction_Text", "TEXT")
]

# التحقق من وجود الحقول وإضافتها إذا كانت غير موجودة
existing_fields = [field.name for field in arcpy.ListFields(polygon_layer)]
for field_name, field_type in fields:
    if field_name not in existing_fields:  # إذا كان الحقل غير موجود
        arcpy.AddField_management(polygon_layer, field_name, field_type)

# استخدام UpdateCursor لتحديث القيم في الحقول الجديدة
with arcpy.da.UpdateCursor(polygon_layer, ["Name", "SHAPE@", "SHAPE@AREA", "SHAPE@LENGTH",
                                           "Area", "Perimeter", "Length", "Length_Direction_Degree",
                                           "Length_Direction_Text", "Width", "Width_Direction_Degree", "Width_Direction_Text"]) as cursor:
    for row in cursor:
        dune_name = row[0]                    # اسم الكثيب
        polygon_geom = row[1]                 # شكل البوليجون
        polygon_area = row[2]                 # مساحة الكثيب
        polygon_perimeter = row[3]            # محيط الكثيب
        polygon_extent = polygon_geom.extent  # الحصول على extent للبوليجون
        width = polygon_extent.width          # عرض المستطيل المحيط
        height = polygon_extent.height        # طول المستطيل المحيط

        # حساب اتجاه الطول (أول نقطتين في المستطيل المحيط)
        points = polygon_geom.getPart(0)
        point1 = points[0]
        point2 = points[1]

        # حساب زاوية اتجاه الطول
        delta_x = point2.X - point1.X
        delta_y = point2.Y - point1.Y
        angle_radians = np.arctan2(delta_y, delta_x)
        angle_degrees = np.degrees(angle_radians)
        length_direction_angle = (angle_degrees + 360) % 360  # زاوية اتجاه الطول

        # تحويل زاوية الطول إلى نص الاتجاهات الثمانية
        length_direction = angle_to_direction(length_direction_angle)
        
        # حساب زاوية اتجاه العرض (زاوية الطول + 90 درجة)
        width_direction_angle = (length_direction_angle + 90) % 360
        width_direction = angle_to_direction(width_direction_angle)

        # تحديث الحقول بالقيم المحسوبة
        row[4] = polygon_area                    # المساحة
        row[5] = polygon_perimeter               # المحيط
        row[6] = height                          # الطول
        row[7] = length_direction_angle          # زاوية اتجاه الطول بالدرجة
        row[8] = length_direction                # اتجاه الطول كنص
        row[9] = width                           # العرض
        row[10] = width_direction_angle          # زاوية اتجاه العرض بالدرجة
        row[11] = width_direction                # اتجاه العرض كنص

        cursor.updateRow(row)  # تحديث الصف بالقيم الجديدة

print("تمت إضافة الحقول (عند الحاجة) وتحديث البيانات بنجاح.")
```

---

### مخرجات الأداة
سيتم تحديث جدول السمات في طبقة البوليغون بالقيم المحسوبة، مما يوفر تفاصيل شاملة حول أبعاد الكثيب واتجاهاته، مثل المساحة، المحيط، الطول، العرض، واتجاه الطول والعرض.
