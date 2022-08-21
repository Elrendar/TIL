# API - Application Programming Interface

[목록으로 돌아가기](/README.md)

## API란?

API는 프로그램과 프로그램이 서로 소통할 수 있게 해주는 존재이다.

서버에서 API를 만들어서 공개하면, 외부에서 해당 API를 사용해 원하는 기능을 수행하거나, 가공된 데이터를 받아갈 수 있다.

## 만들어본 API 예시

내가 쓴 글을 db에 요청하는 api이다.

```Python
@app.route("/api/myposts", methods=["GET"])
def myposts():
    # 쿠키에서 토큰을 가져와서 복호화한 뒤, db에 보내 해당 유저가 작성한 글 목록을 받아온다
    token_receive = request.cookies.get("campuspot_token")

    try:
        payload = jwt.decode(token_receive, SECRET_KEY, algorithms=["HS256"])
        user_info = db.users.find_one({"email": payload["email"]}, {"_id": False})
        my_posts = list(db.posts.find({"email": user_info["email"]}, {'_id': False}))

        return jsonify({'result': 'success', 'my_posts': my_posts})
    except jwt.ExpiredSignatureError:
        return redirect(url_for("login", msg="로그인 시간이 만료되었습니다."))
    except jwt.exceptions.DecodeError:
        return redirect(url_for("login", msg="로그인 정보가 존재하지 않습니다."))
```
