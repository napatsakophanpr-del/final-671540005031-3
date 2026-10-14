# ANSWER — นายนภัสกรพรรณ พรมช่วย 671540005031-3

## Part A — ข้อมูลจาก image kamthornsa/ksu-promo:1.0

| คำถาม | คำตอบ | คำสั่งที่ใช้ |
| --- | --- | --- |
| Port | `3000/tcp` (ใน history มีบรรทัด `EXPOSE [3000/tcp]` ที่ image นี้เพิ่มเอง ส่วน `80/tcp` ติดมาจาก base image และ nginx config ใน image ตั้ง `listen 3000;`) | `docker image inspect kamthornsa/ksu-promo:1.0 --format "{{json .Config.ExposedPorts}}"` และ `docker history --no-trunc kamthornsa/ksu-promo:1.0` |
| CMD | `["nginx", "-g", "daemon off;"]` (รันผ่าน ENTRYPOINT `/docker-entrypoint.sh`) | `docker image inspect kamthornsa/ksu-promo:1.0 --format "{{json .Config.Cmd}} {{json .Config.Entrypoint}}"` |
| Base image | `nginx:alpine` (nginx 1.31.6 บน Alpine 3.24.2: layer ล่างสุดคือ `alpine-minirootfs-3.24.2`, ENV `NGINX_VERSION=1.31.6`, label `NGINX Docker Maintainers` และ layer 8 ชั้นแรกตรงกับ `nginx:alpine`) | `docker history --no-trunc kamthornsa/ksu-promo:1.0` และ `docker image inspect kamthornsa/ksu-promo:1.0 --format "{{json .RootFS.Layers}}"` |

## Part B — รหัสลับบนหน้าโปรโมชั่น

`ESPRESSO-47` (จากหน้า http://localhost:8081 ของ image `kamthornsa/ksu-promo:1.0` ซึ่งแสดง Promo v1)


## Part C — คำถาม

1. หน้าเมนู build จากซอร์สของเราเอง (`index.html` + `Dockerfile`) แก้ไฟล์แล้ว build image ใหม่ก็เปลี่ยนได้ แต่หน้าโปรโมชั่นเป็น image สำเร็จรูปจาก Docker Hub ไฟล์ HTML ถูก COPY ฝังอยู่ใน layer ของ image แล้ว และเราไม่มีซอร์สโค้ด image เป็นแบบ read-only จึงต้องให้เจ้าของ image (ฝ่ายการตลาด) แก้ซอร์สแล้ว build และ push tag ใหม่มาให้ใช้แทน
2. ไม่ได้หน้าเว็บเลย เมื่อไม่ใส่ tag Docker จะใช้ tag `latest` ให้อัตโนมัติ แต่ repo `kamthornsa/ksu-promo` ไม่มี tag `latest` (บน Docker Hub มีแค่ `1.0`) ตอนรัน `docker compose up -d` จึงขึ้น error `docker.io/kamthornsa/ksu-promo:latest: not found` และ service promo ไม่ถูกสร้าง แปลว่าควรระบุ tag ให้ชัดเจนทุกครั้ง จะได้ image เวอร์ชันที่ต้องการแน่นอน
3. `"8081:3000"` ฝั่งซ้ายคือ port บนเครื่องโฮสต์ที่เบราว์เซอร์ใช้เข้า (`localhost:8081`) ฝั่งขวาคือ port ภายใน container ที่โปรแกรมเปิดรอรับอยู่ ฝั่งขวาไม่ใช่ 80 เพราะ nginx ใน image นี้ตั้งค่าไว้ใน `/etc/nginx/conf.d/default.conf` ให้ `listen 3000;` (image นี้ EXPOSE 3000 เอง) ถ้า map ไป 80 ใน container จะไม่มีโปรแกรมรอรับที่ port นั้น จึงเปิดเว็บไม่ได้
4. หน้าเว็บไม่เปลี่ยน เพราะ `index.html` ถูก COPY เข้าไปใน image ตอน build แล้ว คำสั่ง `up -d` จะเห็นว่ามี image `final-...-menu` อยู่แล้วจึงไม่ build ใหม่ และเพราะ config ไม่เปลี่ยน container เดิมจึงรันต่อด้วยไฟล์เก่า คำสั่งที่ถูกต้องคือ `docker compose up -d --build` ซึ่งจะ build image menu ใหม่และสร้าง container menu ใหม่ ส่วน promo ยังรันอยู่ตามเดิม
5. ใช้ `http://localhost:8081` ชื่อ `promo` เป็นชื่อ DNS ที่ใช้ได้เฉพาะภายใน network ของ Compose (container คุยกันเอง) แต่ลิงก์ในหน้าเว็บถูกเปิดโดยเบราว์เซอร์บนเครื่องโฮสต์ ซึ่งอยู่นอก network นั้น จึงแปลงชื่อ `promo` ไม่ได้ และ port 3000 ก็เป็น port ภายใน container ที่โฮสต์เข้าไม่ถึงโดยตรง ต้องเข้าผ่าน port ที่ publish ไว้คือ 8081

## ภาพหน้าจอ

![Checkpoint](screenshots/00-menu.png)
![Menu](screenshots/01-menu.png)
![Promo](screenshots/02-promo.png)
