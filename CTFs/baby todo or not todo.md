---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [IDOR, authorization-bypass, broken-auth]
---
>[!quote]
> I'm so done with these bloody HR solutions coming from those bloody HR specialists, I don't need anyone monitoring my thoughts, or do I... ?****


# Set up

-

# Information Gathering

![Pasted image 20210818202848.png](../../zzz_res/attachments/Pasted_image_20210818202848.png)

```python
from flask import Blueprint, request, jsonify, session, render_template, g
from application.util import verify_integrity
from application.models import todo

main  = Blueprint('main', __name__)
api   = Blueprint('api', __name__)

@main.route('/')
def index():
	context = {
		'list_access': g.user,
		'secret': todo.get_secret_from(g.user)
	}
	return render_template('index.html', **context)

@api.before_request
@verify_integrity
def and_then(): pass

# TODO: There are not view arguments involved, I hope this doesn't break
# the authentication control on the verify_integrity() decorator
@api.route('/list/all/')
def list_all():
	return jsonify(todo.get_all())

@api.route('/list/<assignee>/')
def list_tasks(assignee):
	return jsonify(todo.get_by_user(assignee))

@api.route('/add/', methods=['POST'])
def add():
	todo.add(g.name, g.user)
	return {'success': f'Successfuly added {g.name} by user {g.user}'}

@api.route('/rename/<int:todo_id>/<new_name>/')
def rename_task(todo_id, new_name):
	g.selected.rename(new_name)
	return {'success': f'Successfuly edited {todo_id} to {new_name}'}

@api.route('/delete/<int:todo_id>/', methods=['DELETE'])
def delete(todo_id):
	g.selected.delete()
	return {'success': f'Successfuly deleted {todo_id}'}

@api.route('/complete/<int:todo_id>/')
def complete(todo_id):
	g.selected.complete()
	return {'success': f'Successfuly completed {todo_id}'}

@api.route('/assign/<int:todo_id>/<new_assignee>/')
def assign(todo_id, new_assignee):
	g.selected.reassign(new_assignee)
	return {'success': f'Successfuly reassigned {todo_id} to {new_assignee}'}
```

# Exploitation

![Pasted image 20210818204256.png](../../zzz_res/attachments/Pasted_image_20210818204256.png)

# Flag

>[!success]
>`HTB{l3ss_ch0r3s_m0r3_h4ck1ng...right?!!1}`

