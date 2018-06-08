# Day 3

## 새로운 폴더에 sinatra 프로젝트 넣기

* app.rb 파일 생성 후 views 폴더 생성
* sinatra와 sinatra-reloader 잼을 설치

test_app/app.rb

```ruby
require 'sinatra'
require 'sinatra/reloader'

get '/' do
    erb :app
end
```





<form action="/login" method="POST">와 같이 post방식을 쓰면

루비 파일의 /login을 가진 함수를 get이 아닌 post로 써줘야한다.



그냥 복붙으로 https://synatra-kkmcd.c9users.io/login로 접속하면

get방식으로 접속하기 때문에 오류창이 뜬다.



id/pw 를 검사하는 페이지는 눈에 보이면 안되고, redirection을 통해서 다른 페이지를 요청해야함. 예를들어, 네이버 로그인 후에 네이버 메인창이 뜨는 것과 같은 이치



```ruby
post '/login' do
   if id.eql?(params[:id])
       # 비밀번호를 체크하는 로직
       if pw.eql?(params[:password])
           #erb :complete
           redirect '/complete'
       else
           @msg = "비밀번호가 틀립니다."
           redirect '/error'
       end
       
   else
       # ID가 존재하지 않습니다.
       @msg = "ID가 존재하지 않습니다."
       redirect '/error'
   end
end
```



client -> controller -> view 의 사이클을 돌 때, 하나의 한개의 단만 사용가능하다. 즉, 다음 단계로 넘어가면 그 전의 단계는 disconnection된다. 따라서, 위와같이 에러메시지를 보여주는 것이 아닌 다른 방식으로 해야한다.



```ruby
User.find_id(params[:id]).authenticate(params[:password])
```

```ruby

<p>----form action을 이용한 방법----</p>
<form action="https://search.naver.com/search.naver">
    <input type="text" name="query" placeholder="네이버 검색창">
    <input type="submit">
</form>

<form action="https://www.google.com/search">
    <input type="text" name="q" placeholder="구글 검색창">
    <input type="submit">
</form>

```

위의 경우에는 input text의 name이 query이고, 주소가 "https://search.naver.com/search.naver?query="이기 때문에  query="text의 내용"으로 자동 연결된다.

```ruby

<p>----form method POST를 이용한 방법----</p>
<form method="POST">
    <input type="hidden" name="engine" value="naver">
    <input type="text" name="query" placeholder="네이버 검색">
    <input type="submit" value="검색">
</form>

<form method="POST">
    <input type="hidden" name="engine" value="google">
    <input type="text" name="q" placeholder="구글 검색">
    <input type="submit" value="검색">
</form>
```



```ruby
post '/search' do
    puts params[:engine]
    case params[:engine]
    when "naver"
        redirect "https://search.naver.com/search.naver?query=#{params[:query]}"
    when "google"
        redirect "https://www.google.com/search?q=#{params[:q]}"
    end
end
```



ruby string interpolation





### 문제

1. /op.gg

2. op.gg에서 직접 검색한 결과

3. 승패 수만 보여주기

4. select 태그를 이용해서 두 가지 방법 중 선택하기

5. op.gg                검색            소환사       검색

   승패만 보기

6. 조건 form 태그를 1개 action 1개 혹은 2개

### 답

#### op_gg.erb

```ruby
<form>
    <select name="search_method">
        <option value="self"> 승패만 보기 </option>
        <option value="opgg"> OP.GG에서 보기 </option>
    </select>
    <input type="text" placeholder="소환사 이름" name="userName">
    <input type="submit" value="검색">
</form>

<% if params[:userName] %> <!--눈에 보이지 않아도 되는건 =만 없애면 된다-->
<ul>
    <li><%= params[:userName] %>님의 전적입니다.</li>
    <li><%= @win %> 승</li>
    <li><%= @lose %> 패</li>
</ul>
<% end %>
```



#### app.rb

```ruby
get '/op_gg' do

    if params[:userName]
        case params[:search_method]
        # op.gg에서 승/패 수만 크롤링하여 보여줌
        when "self"
        # RestClient를 이용하여 op.gg에서 검색결과 페이지를 크롤링
        url = RestClient.get(URI.encode("http://www.op.gg/summoner/userName=#{params[:userName]}"))
        # 검색결과 페이지 중에서 win과 lose 부분을 찾음
        result = Nokogiri::HTML.parse(url)
        # nokogiri를 이용하여 원하는 부분을 골라냄
        #css는 태그라던가 클래스라던가를 찾아나감. copy_selector로 찾을 수 있음
        #win = result.css('#GameAverageStatsBox-matches > div.Box > table > tbody > tr:nth-child(1) > td:nth-child(1) > div > span.win').first
        #lose = result.css('#GameAverageStatsBox-matches > div.Box > table > tbody > tr:nth-child(1) > td:nth-child(1) > div > span.lose').first        
        win = result.css('span.win').first
        lose = result.css('span.lose').first
        # 검색 결과를 페이지에서 보여주기 위한 변수 선언
       @win = win.text
       @lose = lose.text
        # html 태그 제거
        # 검색 결과를 op.gg에서 보여줌
        
        when "opgg"
            url = URI.encode("http://www.op.gg/summoner/userName=#{params[:userName]}")
            redirect url
        end
    end
    erb :op_gg
end
```

