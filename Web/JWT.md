# PyJWT를 사용해서 JWT 생성 및 로그인 세션 유지하기

`Javascript: sign_in()`

```javascript
        // 입력된 이메일과 비밀번호를 서버로 전송
        $.ajax({
          // 이 타입과 url이 파이썬쪽 @app.route("url", methods=["타입"])과 일치해야 함
          type: "POST",
          url: "/sign_in",

          // 여기서 보낸 data를 파이썬에서 받아서 처리하고, 다시 이곳으로 돌아온다
          data: {
            email_give: email,
            password_give: password,
          },

          // @app.route("/sign_in", methods=["POST"])
          // def sign_in():

          success: function (response) {
            // 입력한 계정이 db에 존재하면
            if (response["result"] === "success") {
              // 받아온 jwt를 사용해서 세션 쿠키 생성 (브라우저 종료시 사라짐)
              $.cookie("campuspot_token", response["token"], { path: "/" });
              // 메인 페이지로
              window.location.replace("/");
            } else {
              // db에 없으면 경고 메시지
              alert(response["msg"]);
            }
          },
        });
```

`Python` `Flask`

아이디(이메일)와 암호화된 비밀번호를 db에서 검색한 뒤, 있으면 JWT를 생성하여 클라이언트(자바스크립트)에 반환

```Python
@app.route("/sign_in", methods=["POST"])
def sign_in():
    # 이메일과 비밀번호를 넘겨받은 뒤
    email_receive = request.form["email_give"]
    password_receive = request.form["password_give"]
    # 비밀번호를 암호화
    pw_hash = hashlib.sha256(password_receive.encode("utf-8")).hexdigest()
    # db에서 검색
    result = db.users.find_one({"email": email_receive, "password": pw_hash})

    # db에 있으면
    if result is not None:
        # 필요한 정보로 payload를 생성하고
        payload = {
            "email": email_receive,
            # 로그인 24시간 유지
            "exp": datetime.utcnow() + timedelta(seconds=60 * 60 * 24),
        }
        # JWT를 생성해서 반환한다
        # PyJWT 의 v2.0.0부터 jwt.encode()의 반환값이 바뀌어서 .decode()가 필요 없어졌다.
        token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")

        # 이 반환값이 jQuery ajax함수의 success: function (response) < 이 response 안에 담긴다.
        return jsonify({"result": "success", "token": token})
    # 찾지 못하면
    else:
        return jsonify({"result": "fail", "msg": "이메일/비밀번호가 일치하지 않습니다."})
```

로그인 이후 JWT가 유효한지 검사하는 과정

```Python
@app.route("/")
def home():
    # 쿠키에서 토큰을 가져온다
    token_receive = request.cookies.get("campuspot_token")
    try:
        # 서버에 저장된 시크릿키를 사용해 토큰을 복호화한다.
        payload = jwt.decode(token_receive, SECRET_KEY, algorithms=["HS256"])

        # payload에서 계정 정보를 뽑아내 db와 대조한다.
        user_info = db.users.find_one({"email": payload["email"]}, {"_id": False})
        
        # 페이지 렌더
        return render_template("main.html")
    except jwt.ExpiredSignatureError:
        return redirect(url_for("login", msg="로그인 시간이 만료되었습니다."))
    except jwt.exceptions.DecodeError:
        return redirect(url_for("login", msg="로그인 정보가 존재하지 않습니다."))
```
