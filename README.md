# Documents API (Thai PBS APIs)

เอกสารนี้อธิบาย endpoint สำหรับดึง “Posts” ของแต่ละ `site` เพื่อใช้ต่อกับ vendor อื่น

> หมายเหตุ: ในระบบนี้ไม่มีการยืนยันตัวตน (Authentication) และรองรับ CORS (`Access-Control-Allow-Origin: *`) ในการตอบกลับ JSON

Production base URL: `https://api.thaipbs.net`

## Supported sites

รองรับ `site` ต่อไปนี้ 

- `now`
- `news`
- `verify`
- `theactive`
- `decode`
- `world`
- `org`
- `policywatch`

---

## 1) List posts

### `GET /v1/:site/posts`

ดึงรายการโพสต์ของ `site` นั้น ๆ (พร้อม field สำหรับ pagination)

#### Query params

- `page` (optional, integer)
  - default: `1`
- `limit` (optional, integer)
  - default: `20`
- `tag` (optional, string)
  - ใช้กรองตามชื่อแท็ก/แฮชแท็ก
- `section` (optional, string)
  - ใช้กรองหมวด/หมวดข่าว (แล้วแต่ `site`)
- `categoryId` (optional, integer as string)
  - ใช้กรองหมวดด้วย `id` (รองรับหลัก ๆ สำหรับ `now` และ `theactive` และ `decode`)
- `type` (optional, string)
  - ใช้กำหนดชนิดคอนเทนต์ (แล้วแต่ `site`)
  - `verify`:
    - `type=verify` | `type=article`
    - default: `article` (ถ้าไม่ส่ง `type`)
  - `theactive`:
    - ค่า `type` ที่รองรับ (case-insensitive): `news`, `read`, `article`, `view`, `gallery`, `video`, `podcast`, `data`, `data-article`, `data_article`
    - default: `news` (ถ้าไม่ส่ง `type`)
  - `decode`:
    - ค่า `type` ที่รองรับ (case-insensitive): `post`, `listen`, `video`, `de_story`, `event`
    - default: `post` (ถ้าไม่ส่ง `type`)
    - หมายเหตุ: เพื่อความเข้ากันได้ API อาจรับค่ารูปแบบ theactive-like ด้วย เช่น `news/read/article/...` -> `post`, `podcast` -> `listen`
  - `world`:
    - รองรับ `type` ตามรายการนี้ (case-insensitive): `News`, `Program`, `Podcast`, `Gallery`, `Breaking News`, `Quote`, `Infographic`, `Live`
    - รองรับการส่งหลายค่าได้ด้วยเครื่องหมาย `,` เช่น `type=News,Quote`
    - ถ้าไม่ส่ง `type` จะ default อยู่ที่ `News` ฝั่ง list
  - `org`:
    - ค่า `type` ที่รองรับ: `post`, `gallery`, `committee`, `staff-member`, `procurement`, `careers`, `annual_report`, `programs`, `faq`
    - default: `post` (ถ้าไม่ส่ง `type`)
  - `policywatch`:
    - ค่า `type` ที่รองรับ: `article`, `short_clip`, `activity`, `policy`, `localpolicies`, `agenda-policy`, `video`, `live`, `forum`
    - default: `article` (ถ้าไม่ส่ง `type`)

#### Response

ระบบจะตอบกลับ JSON โดยมีอย่างน้อย:

- `data`: array ของรายการ posts (แต่ละองค์ประกอบคือ object หนึ่งรายการ — รายละเอียดด้านล่าง)
- และ `page`, `total`, `minTs`, `Ts` ฯลฯ ตาม `site`

`news` อาจมี field ระดับ root เพิ่มจาก upstream (เช่น `total` ฯลฯ) นอกเหนือจาก `data`

---

##### `data`: array ของรายการ posts

แต่ละ element ใน `data` คือ **สรุปโพสต์หนึ่งรายการในรูปแบบ Object** สำหรับใช้ในลิสต์/หน้าแรก ไม่ใช่เนื้อหาเต็ม

- **ไม่มี `content` (เนื้อหาเต็ม)** ในรายการ list — ดึงเนื้อหาเต็มจาก `GET /v1/:site/posts/:id`
- รูปแบบ **`categories`** และ **`tags`** ใช้โครงสร้างเดียวกันทุก `site` (ที่มีฟิลด์เหล่านี้)

**รูปแบบ Object ที่ใช้ร่วม**

| ชื่อ | ความหมาย |
|------|----------|
| `CategoryItem` | `{ "id": number \| null, "title": string, "slug": string \| null, "url": string \| null }` |
| `TagItem` | `{ "title": string, "url": string \| null }` |
| `AuthorItem` | `{ "id": number \| null, "name": string, "description": string, "url": string \| null, "avatar": string \| null }` |

**ฟิลด์พื้นฐาน (ทุก `site`)**

ตารางนี้คือฟิลด์ที่ **ทุก `site`** ส่งกลับในแต่ละ element ของ `data` (ชื่อเดียวกัน โครงสร้างเดียวกันของ Object) รายละเอียดที่มาของข้อมูลหรือความหมายเฉพาะต้นทางอยู่ในตาราง **ฟิลด์เพิ่มเติมตาม `site`** ด้านล่าง

| Field | คำอธิบาย |
|-------|----------|
| `id` | รหัสโพสต์ (รูปแบบตามต้นทาง — เช่น `number` จาก CMS) |
| `title` | หัวข้อ |
| `categories` | `CategoryItem[]` |
| `tags` | `TagItem[]` |
| `author` | ชื่อผู้เขียนแบบสตริงเดียว |
| `abstract` | บทคัดย่อ / คำโปรย / คำอธิบายสั้น |
| `mediaUrl` | URL รูปปก |
| `media` | `{ "default": string \| null }` |
| `restUrl` | path ของ `GET /v1/:site/posts/:id` ในระบบนี้ |
| `canonical` | URL บนเว็บต้นทาง |
| `publishTime` | ISO datetime |
| `createTime` | ISO datetime |
| `lastUpdateTime` | ISO datetime |
| `viewCount` | จำนวน view (บาง `site` ใน list อาจคงเป็น `0`) |
| `speechPath` | path สำหรับ TTS (ถ้ามี) |
| `seoDetails` | `{ title, slug, description, ogImage, xImage }` |

**ฟิลด์เพิ่มเติมตาม `site` (optional / ไม่ใช่ทุก `site`)**

เฉพาะฟิลด์ที่ **ไม่ได้อยู่ในตารางพื้นฐาน** หรือมีข้อควรทราบเฉพาะ `site`

**`now`**

ไม่มีฟิลด์เพิ่มนอกตารางพื้นฐาน — หมายเหตุที่มา: `categories` สูงสุดหนึ่งหมวดจาก relation ของ Now, `tags` ลิงก์ไป `/tags?q=...`

**`news`**

| Field | คำอธิบาย |
|-------|----------|
| *(อื่น ๆ)* | มี field เพิ่มจาก **onecms** ผ่าน `...restItem` (เช่น `image`, `media` แบบเต็ม ฯลฯ) — **ไม่รวม `content`** (ถูกตัดก่อน merge) |

**`verify`**

หมายเหตุที่มา (ฟิลด์เดียวกับตารางพื้นฐาน): `categories` จาก taxonomy `category-verify`, `tags` จากแฮชแท็ก, `viewCount` จาก meta page view

| Field | คำอธิบาย |
|-------|----------|
| `type` | `verify` หรือ `article` (ตรงกับ query `type`) |
| `verify`, `verification`, `contentCenter`, `guidelines` | เนื้อหาเฉพาะ verify (ไม่ใช่ HTML body ทั้งหมด) |
| `authors` | `AuthorItem[]` |
| `editor` | สตริงชื่อผู้บรรณาธิการ |
| `editors` | `AuthorItem[]` |

**`theactive`**

หมายเหตุที่มา: `categories` จาก taxonomy `category`, `tags` จาก `post_tag`

| Field | คำอธิบาย |
|-------|----------|
| `type` | ชนิดโพสต์ WP (เช่น `news`, `video`) — ตรงกับ query `type` / `item.type` |
| `authors` | `AuthorItem[]` |

**`decode`**

หมายเหตุ: กรอง `section` / `categoryId` ใช้ได้เมื่อ `type` ไม่ใช่ `de_story` — `tags` ใน list อาจเป็น `[]` เมื่อ `type` ไม่ใช่ `post`

| Field | คำอธิบาย |
|-------|----------|
| `type` | `post`, `listen`, `video`, `de_story`, `event` (ตรงกับ query `type`) |
| `authors` | `AuthorItem[]` |

**`org`**

พารามิเตอร์ `type` ใน `GET /v1/org/posts` ใช้เลือก WordPress endpoint — ค่าเดียวกับฟิลด์ `type` ในแต่ละรายการ

| Field | คำอธิบาย |
|-------|----------|
| `type` | ชนิดคอนเทนต์ — ตรงกับ `?type=` และตาราง **ค่า `type`** ด้านล่าง |
| `authors` | `AuthorItem[]` |
| `gallery` | ข้อมูลแกลอรี่จาก `meta_box` (ถ้ามี) — **ต่างจาก** ค่า `type` = `gallery` |

หมายเหตุที่มา `categories`: ถ้า `type` เป็น `gallery` หมวดมาจาก `_embedded['wp:term'][1]` ชนิดอื่นจาก taxonomy `category`

**ค่า `type` (query `type` และฟิลด์ `type` ใน response)**

| ค่า `type` | ความหมาย |
|------------|----------|
| `post` | ข่าว / บทความ — **default** ถ้าไม่ส่ง `type` (WP: `posts`) |
| `gallery` | แกลอรี่ (ชนิดโพสต์) |
| `committee` | คณะกรรมการ |
| `staff-member` | บุคลากร (รองรับ alias `staff_member`, `staff member`) |
| `procurement` | จัดซื้อจัดจ้าง |
| `careers` | รับสมัครงาน (รองรับ `career`) |
| `annual_report` | รายงานประจำปี (รองรับ `annual-report`, `annual report`) |
| `programs` | ติดต่อรายการ (รองรับ `program`) |
| `faq` | คำถามที่พบบ่อย |

**`world`**

หมายเหตุที่มา: `categories` มักเป็นหมวดแรกจาก array หมวด, `tags` ลิงก์ไปค้นหา world

| Field | คำอธิบาย |
|-------|----------|
| `type` | ชนิดคอนเทนต์จาก Strapi (เช่น `News`) |
| `authors` | `AuthorItem[]` |
| `embed` | embed live / embed อื่น ๆ (ถ้ามี) |

**`policywatch`**

หมายเหตุที่มา: `categories` จาก taxonomy `policy-cat`, `tags` จาก taxonomy `hashtag`

| Field | คำอธิบาย |
|-------|----------|
| `type` | ชนิดคอนเทนต์ WP (`article`, `short_clip`, `activity`, `policy`, `localpolicies`, `agenda-policy`, `video`, `live`, `forum`) |
| `authors` | `AuthorItem[]` |

---

**Pagination ระดับ response (นอก `data`)**

สำหรับหลาย `site` จะมี field ประมาณ:

- `page` — หน้าปัจจุบัน
- `total` — จำนวนรายการทั้งหมด (หรือหลังกรอง แล้วแต่ `site`)
- `totalPages` — จำนวนหน้า
- `minTs`, `Ts` — timestamp (unix) สำหรับ sync ลิสต์

`news` รวม field เหล่านี้เข้ากับ `...result` จาก upstream

#### Example

List `now`

```bash
curl "https://api.thaipbs.net/v1/now/posts?page=1&limit=20&tag=PM2.5&categoryId=8"
```

List `verify` (เลือก Type)

```bash
curl "https://api.thaipbs.net/v1/verify/posts?type=verify&tag=%E0%B8%AD%E0%B8%B4%E0%B8%AB%E0%B8%A3%E0%B9%88%E0%B8%B2%E0%B8%99&page=1&limit=20"
```

List `theactive` (เลือก Type + section)

```bash
curl "https://api.thaipbs.net/v1/theactive/posts?type=video&section=Pollution&page=1&limit=20"
```

List `decode` (เลือก Type + section)

```bash
curl "https://api.thaipbs.net/v1/decode/posts?type=video&section=Economy&page=1&limit=20"
```

List `world` (ส่งหลาย type)

```bash
curl "https://api.thaipbs.net/v1/world/posts?type=News,Quote&tag=Climate&page=1&limit=20"
```

List `org` (เลือก Type)

```bash
curl "https://api.thaipbs.net/v1/org/posts?type=committee&page=1&limit=20"
```

List `policywatch` (เลือก Type)

```bash
curl "https://api.thaipbs.net/v1/policywatch/posts?type=article&page=1&limit=20"
```

---

## 2) Get a post by id

### `GET /v1/:site/posts/:id`

ดึงรายละเอียดโพสต์แต่ละรายการ

#### Query params

- `type` (optional)
  - `verify`: ใช้เลือก `verify` vs `article` (default = `article`)
  - `theactive`: ใช้เลือก Type ตาม mapping ของ `theactive` (default = `news`)
  - `decode`: ใช้เลือก Type ตาม WP post types: `post`, `listen`, `video`, `de_story`, `event` (default = `post`)
  - `org`: ใช้เลือก Type ตาม WP post types: `post`, `gallery`, `committee`, `staff-member`, `procurement`, `careers`, `annual_report`, `programs`, `faq` (default = `post`)
  - `policywatch`: ใช้เลือก Type ตาม WP post types: `article`, `short_clip`, `activity`, `policy`, `localpolicies`, `agenda-policy`, `video`, `live`, `forum` (default = `article`)
  - `world`: ส่ง `type` ได้ (optional) เพื่อช่วยให้ query filter ตรงกับชนิดคอนเทนต์

#### Response

จะตอบกลับ JSON ของ post หนึ่งรายการ โดย field ที่เจอได้บ่อย (อาจต่างกันเล็กน้อยตาม `site`):

- `id`
- `title`
- `abstract` หรือ `content`
- `categories`: array ของหมวดหมู่
- `tags`: array ของแท็ก
- `author`
- `image` / `imgUrl` / `media.default` (ใช้รูปปก)
- `publishTime`, `createTime`, `lastUpdateTime` (ถ้ามี)
- `viewCount` (ถ้ามี)
- `speechPath` (path สำหรับ TTS)
- `restUrl`: URL กลับไปยัง endpoint detail ของ `posts`
- `canonical`: URL canonical ของบทความ/โพสต์
- `seoDetails`: object (เช่น title/slug/description/ogImage/xImage) (ถ้ามี)
- บาง `site` อาจมี `recommendedNews`, `embed`, `authorInfo`, ฯลฯ

#### Example

Detail `verify` (เลือก Type)

```bash
curl "https://api.thaipbs.net/v1/verify/posts/10416?type=verify"
```

Detail `theactive`

```bash
curl "https://api.thaipbs.net/v1/theactive/posts/86665?type=gallery"
```

Detail `decode`

```bash
curl "https://api.thaipbs.net/v1/decode/posts/77069"
```

Detail `world`

```bash
curl "https://api.thaipbs.net/v1/world/posts/60847"
```

Detail `org`

```bash
curl "https://api.thaipbs.net/v1/org/posts/53942?type=post"
```

Detail `policywatch`

```bash
curl "https://api.thaipbs.net/v1/policywatch/posts/38608?type=article"
```

---

