---
layout: post
title: python发邮件
category: python
keywords:
---

```python
import smtplib, email

sender = 'chenfy@keyou.cn'
receiver = ['chenfy@keyou.cn', 'chenfy27@qq.com']

with open('/etc/passwd') as f:
	msg = email.mime.Text.MIMEText(f.read(), 'plain', 'utf-8')
msg['from'] = sender
msg['to'] = ','.join(receiver)
msg['subject'] = 'passwd'

mail = smtplib.SMTP('smtp.qiye.163.com')
mail.ehlo()
mail.starttls()
mail.login('chenfy@keyou.cn', '62960909')

mail.sendmail(sender, receiver, msg.as_string())
mail.quit()
```
