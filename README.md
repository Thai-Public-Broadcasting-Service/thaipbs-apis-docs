# Documents API (Thai PBS APIs)

เอกสารนี้อธิบาย endpoint สำหรับดึง “Posts” ของแต่ละ `site` เพื่อใช้ต่อกับ vendor อื่น

> หมายเหตุ: ในระบบนี้ไม่มีการยืนยันตัวตน (Authentication) และรองรับ CORS (`Access-Control-Allow-Origin: *`) ในการตอบกลับ JSON

Production base URL: `https://api.thaipbs.net`

## Supported sites

รองรับ `site` ต่อไปนี้ 

- `now`
- `news`
- `locals`
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
- `tag` (optional, string | string[])
  - ใช้กรองตามชื่อแท็ก/แฮชแท็ก
  - รองรับการส่งหลายค่าได้ 2 รูปแบบ:
    - comma-separated: `tag=AI,เทคโนโลยี,อวกาศ`
    - repeated params: `tag=AI&tag=เทคโนโลยี&tag=อวกาศ`
  - เมื่อส่งหลายค่า ระบบจะคืนรายการที่มีแท็กตรงกับ **อย่างน้อยหนึ่งค่า** (OR)
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
| `TagItem` | `{ "title": string, "slug"?: string \| null, "url": string \| null }` |
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

ไม่มีฟิลด์เพิ่มนอกตารางพื้นฐาน — หมายเหตุที่มา: `categories` สูงสุดหนึ่งหมวดจาก relation ของ Now (มี `slug`/`url` เป็น canonical path หมวด เช่น `/now/categories/{slug}`), `tags` ลิงก์ไป `/tags?q=...`

**`news`**

| Field | คำอธิบาย |
|-------|----------|
| *(อื่น ๆ)* | มี field เพิ่มจาก **onecms** ผ่าน `...restItem` (เช่น `image`, `media` แบบเต็ม ฯลฯ) — **ไม่รวม `content`** (ถูกตัดก่อน merge) |

**`locals`**

หมายเหตุที่มา: ข้อมูลจาก `locals` (`strapi`) โดย `restUrl` จะเป็น **slug-only** ในรูป `/v1/locals/posts/:slug`, `canonical` ของโพสต์เป็น `/locals/contents/{slug}`, `tags` มี `slug` แบบ WordPress-style และ `tags.url` เป็น `/locals/contents?tagId={id}` และมี `zones` (รูปแบบเดียวกับ categories/tags) โดย `zones.url` เป็น `/locals/contents?zoneId={id}`

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

List `now` (หลาย tag แบบ comma-separated)

```bash
curl "https://api.thaipbs.net/v1/now/posts?page=1&limit=20&tag=AI,เทคโนโลยี,อวกาศ,วิทยาศาสตร์,เทรนด์,โลกและสิ่งแวดล้อม,Thai%20Genius"
```

List `news` (หลาย tag แบบ repeated params)

```bash
curl "https://api.thaipbs.net/v1/news/posts?page=1&limit=20&tag=AI&tag=เทคโนโลยี&tag=อวกาศ"
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

List `locals`

```bash
curl "https://api.thaipbs.net/v1/locals/posts?page=1&limit=20&tag=PM2.5"
```

---

## 2) Get a post by id / slug

### `GET /v1/:site/posts/:id`

ดึงรายละเอียดโพสต์แต่ละรายการ

> สำหรับ `locals` ค่าพารามิเตอร์ `:id` จะต้องเป็น **slug** (ไม่ใช่เลข id) เช่น `/v1/locals/posts/wwlj8ohrpo0h2zoaxexfro8y-...`

#### Query params

- `type` (optional)
  - `verify`: ใช้เลือก `verify` vs `article` (default = `article`)
  - `theactive`: ใช้เลือก Type ตาม mapping ของ `theactive` (default = `news`)
  - `decode`: ใช้เลือก Type ตาม WP post types: `post`, `listen`, `video`, `de_story`, `event` (default = `post`)
  - `org`: ใช้เลือก Type ตาม WP post types: `post`, `gallery`, `committee`, `staff-member`, `procurement`, `careers`, `annual_report`, `programs`, `faq` (default = `post`)
  - `policywatch`: ใช้เลือก Type ตาม WP post types: `article`, `short_clip`, `activity`, `policy`, `localpolicies`, `agenda-policy`, `video`, `live`, `forum` (default = `article`)
  - `world`: ส่ง `type` ได้ (optional) เพื่อช่วยให้ query filter ตรงกับชนิดคอนเทนต์
  - `locals`: ไม่ใช้ query พิเศษ (slug อยู่ใน path)

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

## 2.1) News gallery

แกลอรี่ภาพภายใต้ **`/v1/news/gallery`** แยกจาก `GET /v1/news/posts` — ใช้สำหรับลิสต์แกลอรี่ กรองหมวด และดึงรายละเอียดพร้อมรูปหลายใบ

> ใน response ระบบจะตั้ง **`restUrl`** ให้ชี้มาที่ API นี้ (ลิสต์และรายละเอียดแกลอรี่) และในรายละเอียดจะตั้ง **`relatedNews[].restUrl`** ของโพสต์ข่าวให้เป็น `GET /v1/news/posts/:id`

### `GET /v1/news/gallery/categories`

รวมหมวดแกลอรี่ที่พบจากการไล่รายการแกลอรี่ (pagination `page` + `limit=100` ต่อหน้า สูงสุด 20 หน้า) แล้ว dedupe ตาม `categoryId`

#### Response

- `minTs`: unix timestamp
- `total`: จำนวนหมวด
- `data`: array ของ `{ "id", "title", "slug", "url" }`
  - `id`: string (รหัสหมวด) — ใช้กับ `categoryId` ใน `GET /v1/news/gallery`
  - `slug`: slug ภาษาอังกฤษของหมวด (ถ้ามี)
  - `url`: path สำเร็จรูปไปลิสต์แกลอรี่ในหมวดนั้น เช่น `/v1/news/gallery?categoryId=...`

#### Example

```bash
curl "https://api.thaipbs.net/v1/news/gallery/categories"
```

### `GET /v1/news/gallery`

ดึงรายการแกลอรี่ รองรับการโหลดเพิ่มและกรองหมวด

#### Query params

- `categoryId` (optional, string) — กรองหมวดแกลอรี่ เช่น `6828b35a5b95c43d673475e5`
- `lastId` หรือ `lastid` (optional) — ใช้โหลดเพิ่ม (load more)
- `limit` (optional) — จำนวนรายการต่อครั้ง (เช่น `10`, `50`, `100`)
- `page` (optional, integer) — แบ่งหน้า (เช่น `page=2` ใช้คู่กับ `limit`)

#### Response

มีฟิลด์เช่น `minTs`, `currentTime`, `currentCategory`, `currentCategoryId`, `total`, `items` แต่ละ element ใน `items` จะมี **`restUrl`** เป็น path ภายใน API นี้: `/v1/news/gallery/{id}`

#### Example

```bash
curl "https://api.thaipbs.net/v1/news/gallery?limit=20"
curl "https://api.thaipbs.net/v1/news/gallery?categoryId=6828b35a5b95c43d673475e5&lastId=331&limit=50"
curl "https://api.thaipbs.net/v1/news/gallery?page=2&limit=20"
```

### `GET /v1/news/gallery/:id`

ดึงรายละเอียดแกลอรี่หนึ่งชุด (เนื้อหา intro, รูปใน `items`, แท็ก, `relatedNews` ฯลฯ)

#### Response

- โครงสร้าง JSON ตามข้อมูลแกลอรี่ (หัวข้อ เนื้อหา รูป แท็ก ข่าวที่เกี่ยวข้อง ฯลฯ)
- เพิ่ม/แทนที่ **`restUrl`** ระดับ root เป็น `/v1/news/gallery/{id}`
- **`relatedNews`**: แต่ละรายการที่เป็นโพสต์ข่าวจะได้ `restUrl` เป็น `/v1/news/posts/{id}`

#### Example

```bash
curl "https://api.thaipbs.net/v1/news/gallery/1251"
```

---

## 3) Policywatch Policies (special)

### `GET /v1/policywatch/policies`

ดึงรายการ Policy ของ `policywatch` โดยใช้ post type `policy` โดยตรง

#### Query params

- `page` (optional, integer, default `1`)
- `limit` (optional, integer, default `20`)
- `tag` (optional) กรองจาก taxonomy `hashtag`
- `section` (optional) กรองจาก taxonomy `policy-cat`
- `includeContent` (optional: `1`/`true`) ให้ส่ง `content` ใน list

#### Response

โครงหลักจะเหมือน list `posts` ของ policywatch และเพิ่มฟิลด์พิเศษ:

- `slug`
- `acf` (object จาก upstream เพื่อเก็บ field พิเศษทั้งหมด)

#### Example

```bash
curl "https://api.thaipbs.net/v1/policywatch/policies?page=1&limit=20&section=economy"
```

### `GET /v1/policywatch/policies/:slug`

ดึงรายละเอียด Policy รายชิ้นด้วย `slug` (ไม่ใช้ id)

#### Response

โครงหลักจะเหมือน detail `posts` ของ policywatch และเพิ่มฟิลด์:

- `slug`
- `acf` (object จาก upstream)

#### Example

```bash
curl "https://api.thaipbs.net/v1/policywatch/policies/finance-101"
```

---

## 4) Categories

### `GET /v1/:site/categories`

ดึงรายการหมวดหมู่ของแต่ละ `site` (รองรับปัจจุบัน: `news`, `now`, `locals`, `verify`, `theactive`, `decode`, `world`, `org`, `policywatch`)

#### Query params

- `page` (optional, integer)
  - default: `1`
- `limit` (optional, integer)
  - default: `20`
- `id` (optional, integer as string)
  - กรอง category ที่มี `id` ตรงกับค่าที่ส่ง — ถ้าเจอจะตอบกลับเป็น `data` ที่มี 1 รายการ, ถ้าไม่เจอ `data` เป็น array ว่าง
  - เมื่อส่ง `id` จะไม่ทำ pagination (`page` = 1, `total` = 0 หรือ 1)
- `slug` (optional, string)
  - กรอง category ที่มี `slug` (case-insensitive) ตรงกับค่าที่ส่ง — ถ้าไม่เจอจาก `slug` จะลอง fallback เทียบกับ `title` ให้อัตโนมัติ
  - เมื่อส่ง `slug` จะไม่ทำ pagination เช่นเดียวกัน
  - หากส่งทั้ง `id` และ `slug` ระบบจะถือว่า match เมื่อ "ตรงอย่างใดอย่างหนึ่ง" (`OR`)

#### Response

ตอบกลับรูปแบบ pagination เดียวกับ posts:

- `minTs`
- `Ts`
- `page`
- `total`
- `totalPages`
- `data`

โดยแต่ละรายการใน `data` มีฟิลด์:

- `id`
- `title`
- `slug`
- `canonical`

หมายเหตุ: `canonical` ใน endpoint categories จะคืนเป็น path (ไม่มี domain) เช่น `locals` จะเป็น `/locals/contents?categoryId={id}`

#### Example

```bash
curl "https://api.thaipbs.net/v1/news/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/now/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/locals/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/verify/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/theactive/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/decode/categories?page=1&limit=10"
```

```bash
curl "https://api.thaipbs.net/v1/world/categories?page=1&limit=10"
```

หมายเหตุ: สำหรับ `world` ค่า `canonical` ของ categories จะเป็นรูปแบบ `/{slug}`

```bash
curl "https://api.thaipbs.net/v1/org/categories?page=1&limit=10"
```

หมายเหตุ: สำหรับ `org` ค่า `canonical` ของ categories จะเป็นรูปแบบ `/org/category/{slug}`

```bash
curl "https://api.thaipbs.net/v1/policywatch/categories?page=1&limit=10"
```

หมายเหตุ: สำหรับ `policywatch` endpoint นี้ใช้หมวดของบทความ (`article-cat`) และค่า `canonical` จะเป็น `/category/{slug}`

หมายเหตุ: สำหรับ `verify` ค่า `canonical` ของ categories แยกตามชนิดหมวด:
- `category-verify` -> `/verify/category/{slug}`
- `category-article` -> `/verify/article/{slug}`

ตัวอย่างกรองด้วย `slug` หรือ `id`:

```bash
curl "https://api.thaipbs.net/v1/theactive/categories?slug=news"
```

```bash
curl "https://api.thaipbs.net/v1/theactive/categories?id=18"
```

### `GET /v1/:site/categories/:id`

ดึง category รายการเดียวด้วย path param

- `:id` รับเป็นได้ทั้ง **id ตัวเลข** หรือ **slug** (รองรับ slug ภาษาไทย / percent-encoded)
- ถ้า `:id` เป็นตัวเลข ระบบจะลองหาด้วย `id` ก่อน, หากไม่เจอจะลองตีความเป็น `slug` ให้อัตโนมัติ
- ถ้าไม่เป็นตัวเลขจะถือว่าเป็น `slug` ตรง ๆ

#### Response

- ถ้าเจอ → ตอบกลับเป็น object ของ category รายการเดียว `{ id, title, slug, canonical }`
- ถ้าไม่เจอ → `404 { "message": "Category not found" }`

#### Example

```bash
curl "https://api.thaipbs.net/v1/theactive/categories/18"
```

```bash
curl "https://api.thaipbs.net/v1/theactive/categories/news"
```

```bash
curl "https://api.thaipbs.net/v1/policywatch/categories/economy"
```

---

## 5) Tags

### `GET /v1/:site/tags`

ดึงรายการแท็ก/แฮชแท็กของแต่ละ `site` (รองรับปัจจุบัน: `news`, `now`, `locals`, `verify`, `theactive`, `decode`, `world`, `org`, `policywatch`)

#### Query params

- `page` (optional, integer)
  - default: `1`
- `limit` (optional, integer)
  - default: `20`
- `id` (optional, integer as string)
  - กรอง tag ที่มี `id` ตรงกับค่าที่ส่ง — `data` จะมี 0 หรือ 1 รายการ
  - ใช้กับ `news`, `world` ไม่ได้ (ต้นทางไม่มี id ของ tag)
- `slug` (optional, string)
  - กรอง tag ที่มี `slug` ตรงกับค่าที่ส่ง — สำหรับ site ที่ไม่มี `slug` แท้ ๆ ระบบจะ fallback ไปจับคู่กับ `title` (case-insensitive) ให้อัตโนมัติ

#### Response

โครงสร้างเดียวกับ categories:

- `minTs`, `Ts`, `page`, `total`, `totalPages`, `data`

โดยแต่ละรายการใน `data` มีฟิลด์:

- `id` (อาจเป็น `null` สำหรับ `news`/`world`)
- `title`
- `slug` (อาจเป็น `null` สำหรับชื่อภาษาไทยที่ slugify แล้วเป็นค่าว่าง)
- `canonical` (path; ไม่มี domain ยกเว้น `world` ซึ่งใช้ search URL เต็ม)

#### หมายเหตุที่มาของแท็ก ต่อ `site`

| Site | ต้นทาง | URL ของ tag (`canonical`) |
|------|--------|----------------------------|
| `news` | รวมจากแท็กในข่าวล่าสุดของ onecms | `/tags?q={tag}` |
| `now` | Strapi `/api/tags` (paginate ผ่าน `pagination[page]`/`pagination[pageSize]`) | `/now/tags/{slug}` |
| `locals` | Strapi `/strapi/api/contents/{documentId}` รวมจาก content ล่าสุด (endpoint `/api/tags` ของ Strapi ถูกซ่อน) | `/locals/contents?tagId={id}` |
| `verify` | WordPress taxonomy `hashtag` ของ verify | `/verify/tag/{name}` |
| `theactive` | WordPress taxonomy `post_tag` ของ theactive | `/tag/{slug}` |
| `decode` | WordPress taxonomy `post_tag` ของ decode | `/tag/{slug}/` |
| `world` | รวมจากฟิลด์ `tag` บน content ล่าสุดของ world (ต้นทางไม่มี endpoint master tag) | `https://world.thaipbs.or.th/search?search={tag}` |
| `org` | WordPress taxonomy `post_tag` ของ org | `/tag/{slug}/` |
| `policywatch` | WordPress taxonomy `hashtag` ของ policywatch | `/tag/{slug}/` |

> สำหรับ site ที่ต้องอ่านเป็น sample (`news`, `locals`, `world`) ระบบจะดึงจาก content ล่าสุดและ deduplicate ก่อนทำ pagination ใน-memory ดังนั้น `total` จะสะท้อนเฉพาะ tag ที่อยู่ใน sample เท่านั้น

#### Example

```bash
curl "https://api.thaipbs.net/v1/theactive/tags?page=1&limit=20"
```

```bash
curl "https://api.thaipbs.net/v1/verify/tags?page=2&limit=50"
```

```bash
curl "https://api.thaipbs.net/v1/news/tags?limit=30"
```

ตัวอย่างกรองด้วย `slug` หรือ `id`:

```bash
curl "https://api.thaipbs.net/v1/decode/tags?slug=ai"
```

```bash
curl "https://api.thaipbs.net/v1/now/tags?id=1"
```

### `GET /v1/:site/tags/:id`

ดึง tag รายการเดียวด้วย path param

- `:id` รับเป็นได้ทั้ง **id ตัวเลข** หรือ **slug**
- ถ้า `:id` เป็นตัวเลข ระบบจะลองหาด้วย `id` ก่อน, หากไม่เจอจะลองตีความเป็น `slug` ให้อัตโนมัติ
- ถ้าไม่เป็นตัวเลขจะถือว่าเป็น `slug` ตรง ๆ

#### Response

- ถ้าเจอ → ตอบกลับเป็น object ของ tag รายการเดียว `{ id, title, slug, canonical }`
- ถ้าไม่เจอ → `404 { "message": "Tag not found" }`

#### Example

```bash
curl "https://api.thaipbs.net/v1/verify/tags/3000"
```

```bash
curl "https://api.thaipbs.net/v1/theactive/tags/climate"
```

```bash
curl "https://api.thaipbs.net/v1/now/tags/Facebook"
```

