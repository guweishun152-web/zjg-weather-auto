name: 张家港每日7:30天气推送

on:
  schedule:
    - cron: '30 23 * * *'  # 北京时间 7:30
  workflow_dispatch:

jobs:
  push_weather:
    runs-on: ubuntu-latest
    steps:
      - name: 获取张家港天气 + 推送微信
        run: |
          # 1. 获取微信 access_token
          ACCESS_TOKEN=$(curl -s "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${{ secrets.WX_APPID }}&secret=${{ secrets.WX_APPSECRET }}" | jq -r .access_token)
          
          # 2. 获取张家港天气
          WEATHER_RESP=$(curl -s "https://devapi.qweather.com/v7/weather/now?location=101190209&key=${{ secrets.WEATHER_KEY }}")
          WEATHER=$(echo $WEATHER_RESP | jq -r .now.text)
          TEMP=$(echo $WEATHER_RESP | jq -r .now.temp)
          WIND=$(echo $WEATHER_RESP | jq -r .now.windDir)
          
          # 3. 获取所有关注者 openid（测试号直接用 @all 推送）
          OPENID_LIST=$(curl -s "https://api.weixin.qq.com/cgi-bin/user/get?access_token=$ACCESS_TOKEN" | jq -r .data.openid[])
          
          # 4. 给每个关注者发模板消息
          for OPENID in $OPENID_LIST; do
            curl -s -X POST "https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=$ACCESS_TOKEN" \
            -H "Content-Type: application/json" \
            -d '{
              "touser": "'"$OPENID"'",
              "template_id": "IneI8E0iNxQeJZgR2h201cY6gR5cU52V5c0V520",
              "data": {
                "thing1": {"value": "张家港早报☀️"},
                "phrase2": {"value": "'"$WEATHER"'"},
                "number3": {"value": "'"$TEMP"℃'"},
                "thing4": {"value": "风向：'"$WIND"'"}
              }
            }'
          done
