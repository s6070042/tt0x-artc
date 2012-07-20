﻿วิธีที่ง่ายที่สุดในการทำ TDD บน Python คือ ใช้ module `doctest` แล้วแนบคำสั่งที่ต้องการทดสอบพร้อมผลลัพท์ (เขียนรูปแบบเดียวกับการสั่งงาน Python shell) ลงไปที่ docstring ของฟังก์ชัน เช่นนี้

	def say_hi(name):
		'''
		>>> say_hi('World')
		Hello World!
		>>> say_hi('Robert')
		Hi Robert!
		'''

		print('Hi {}!'.format(name))

	if __name__ == "__main__":
		import doctest
		doctest.testmod()

เซฟเป็นไฟล์ `to_test.py` แล้วเรียกโปรแกรมนี้ผ่าน terminal จะเห็นผลลัพท์ดังนี้

	$ python to_test.py
	**********************************************************************
	File "to_test.py", line 3, in __main__.say_hi
	Failed example:
		say_hi('World')
	Expected:
		Hello World!
	Got:
		Hi World!
	**********************************************************************
	1 items had failures:
	   1 of   2 in __main__.say_hi
	***Test Failed*** 1 failures.

ซึ่งหมายความว่า test เราไม่ผ่านอยู่ 1 test case เมื่อส่งข้อความ `'World'` ให้กับฟังก์ชั่น คราวนี้ลองปรับปรุงบรรทัดที่สั่ง `print` ให้เป็น

		print('Hello' if name=='World' else 'Hi', '{}!'.format(name))

เมื่อสั่ง `$ python to_test.py` อีกรอบ จะไม่ได้ค่าอะไรคืนมาแล้ว ซึ่งก็คือ test ผ่านหมดนั่นเองครับ

หรือจะขอดูรายงานแบบเต็มๆ ก็ย่อมได้ โดยการเพิ่ม argument `-v` เข้าไปเช่นนี้

	$ python to_test.py -v
	Trying:
		say_hi('World')
	Expecting:
		Hello World!
	ok
	Trying:
		say_hi('Robert')
	Expecting:
		Hi Robert!
	ok
	1 items had no tests:
		__main__
	1 items passed all tests:
	   2 tests in __main__.say_hi
	2 tests in 2 items.
	2 passed and 0 failed.
	Test passed.

---

สำหรับส่วน docstring เราสามารถแนบคำอธิบายอื่นๆ ลงไปกับ test ได้ โดยต้องเว้นบรรทัดว่างเอาไว้เพื่อบอกให้รู้ว่าส่วนไหนที่หมด output จาก test แล้ว และสามารถเขียนส่วน input test หลายบรรทัดได้ สำหรับกรณี error สามารถใส่ `...` ตามตัวอย่าง เพื่อบอกว่าไม่ต้องสนใจ output ส่วนที่เป็นรายงานอย่างละเอียดก็ได้ครับ

	def say_hi(*names):
		r'''
		Says hi to everyone, except world.

		>>> say_hi('World')
		Hello World!
		>>> say_hi('Robert')
		Hi Robert!
		>>> say_hi(
		...         'Batman',
		...         'Joker',
		... )
		... 
		Hi Batman!
		Hi Joker!

		Raise exception if no argument given.
		>>> say_hi()
		Traceback (most recent call last):
		  ...
		Warning: No one to greet!
		'''

		if not names:
			raise Warning('No one to greet!')

		for name in names:
			print('Hello' if name=='World' else 'Hi', '{}!'.format(name))

	if __name__ == "__main__":
		import doctest
		doctest.testmod()

นอกจากนี้ ใน production code จริง การเรียกฟังก์ชัน `doctest.testmod()` ลงไปอาจไม่เหมาะสมนัก เราสามารถยกส่วนนั้นทิ้งไปได้ (เหลือแต่ code เปล่าๆ กับ docsting ที่มี test case อยู่) แล้วสั่งผ่าน shell เช่นนี้แทน

	$ python -m doctest to_test.py

---

หรือถ้าส่วน test case นั้นใหญ่มากจนไม่ควรจะไปอยู่กับ docstring เราอาจยก test case มาเขียนเป็นไฟล์ใหม่ที่บรรจุเฉพาะข้อมูล test ไปเลยก็ได้ ดังนี้

ไฟล์ `to_test.py`

	def say_hi(*names):
		'''Says hi to everyone, except world.'''

		if not names:
			raise Warning('No one to greet!')

		for name in names:
			print('Hello' if name=='World' else 'Hi', '{}!'.format(name))

ไฟล์ `info_test.txt`

	Test Module
	===========

		>>> from to_test import *

	Test Function ``say_hi``
	------------------------

	``say_hi`` is a function that will greet everyone with 'Hi'.

		>>> say_hi('Robert')
		Hi Robert!

	``say_hi`` can also greet many people at the same time.

		>>> say_hi(
		...         'Batman',
		...         'Joker',
		... )
		... 
		Hi Batman!
		Hi Joker!

	But ``say_hi`` would do traditional 'Hello World!' when met World.

		>>> say_hi('World')
		Hello World!

	Lastly, ``say_hi`` will talk no word when no one is listening.

		>>> say_hi()
		Traceback (most recent call last):
		  ...
		Warning: No one to greet!

เรียกผ่าน shell ไม่ต่างจากเดิม เพียงแค่เปลี่ยน argument เป็นไฟล์ที่เขียน test เอาไว้แค่นั้นครับ

	$ python -m doctest info_test.txt
