# 下一个假期

此项目为**下一个假期**小程序（见下方小程序码）以及 [xiayigejiaqi.com](https://xiayigejiaqi.com/) 所使用的配置文件。`jiaqi.json` 为目前最新配置，其它为往年备份。
<div>
<img src="https://general.lesignstatic.com/wechat/qr/jiaqi.jpg" width="200" alt="下一个假期小程序码" />
</div>

## Yearly update

The State Council publishes next year's holiday schedule around November. Once it is out, do the three steps below. Steps 2 and 3 only need step 1 to be pushed.

### 1. Update `jiaqi.json`

1. Archive the current config: `cp jiaqi.json <its year>.json`.
2. Edit `jiaqi.json` for the new year:
   - `year`: the new year.
   - `vacation`: one entry per holiday, in date order.
     - `holiday` is the festival day itself and `start` / `end` the days off (inclusive). `occupied` lists the 补班 make-up workdays (`[]` if none).
     - `name` is the Chinese name and `keywords` the aliases people type (`五一`, `十一`, …).
     - Then append next year's 元旦 and 春节 (正月初一) with `holiday`, `enName`, `name`, `keywords` and `year` only. Their schedule is not out yet, so leave out `start` / `end` / `occupied`.
   - `plans`: keyed by month index `"0"`–`"11"` (January is `"0"`), by the month the plan's holiday falls in. A January plan may take leave in the previous December.
     - A month with a leave plan has `names` (e.g. `"请 4 休 9"`) and `days`, which holds the leave dates for each name in the same order.
     - A month without one has a single `emoji` that looks disappointed.
     - In 请 X 休 Y, X is the number of leave days and Y the run of consecutive days off they give (weekends + the holiday + the leave days). A 补班 day that the plan takes off counts as a leave day. Count Y on a calendar before saving.
   - `version`: the current time in ms (`node -e 'console.log(Date.now())'`).
3. Check that it parses: `python3 -m json.tool jiaqi.json > /dev/null`.
4. Commit and push to `master`.

### 2. Publish the remote config

The mini program and the website read `https://general.lesignstatic.com/config/jiaqi.json`. That is a Qiniu CDN in front of the AWS Beijing (cn-north-1) S3 bucket `general.lesignstatic.com`, whose origin URL is `https://s3.cn-north-1.amazonaws.com.cn/general.lesignstatic.com/config/jiaqi.json`.

1. In the AWS China console, upload `jiaqi.json` to `config/` in the S3 bucket `general.lesignstatic.com`, replacing the old file.
2. In the Qiniu console (CDN → 刷新预取), refresh the URL `https://general.lesignstatic.com/config/jiaqi.json`.
3. Check that the origin and the CDN both return the new `year` and `version`:

   ```bash
   for u in https://s3.cn-north-1.amazonaws.com.cn/general.lesignstatic.com/config/jiaqi.json https://general.lesignstatic.com/config/jiaqi.json; do curl -s "$u" | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d["year"], d["version"])'; done
   ```

### 3. Update the 公众号 auto-reply

The 下一个假期 公众号 answers 「假期」「春节」「国庆」… from two files in `lesign/wechat-channel`. `consts/jiaqi.json` holds this year's holidays, keyed by name and keyword. `consts/jiaqi-next.json` holds next year's 元旦 / 春节.

1. In `wechat-channel`, run `bun scripts/update-jiaqi-online.js`. It reads `jiaqi.json` from this repo's `master` on GitHub and rewrites both files. It also adds `除夕` (the 春节 entry, dated the day before) to each file.
2. Check a few replies: `TZ=Asia/Shanghai node -e "console.log(require('./utils/jiaqi').jiaqi('假期'))"`.
3. Commit (a husky hook runs prettier) and push `master`.
4. Deploy on the `bj` server, where pm2 runs it as process `studio` (details in `DEV/servers/Agent.md`): `git pull --ff-only origin master && pm2 restart studio`. Then send 「假期」 to the 公众号 and check the reply.

---

Made by [leplay](https://leplay.net/).
