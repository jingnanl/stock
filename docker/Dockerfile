# 基础镜像
# https://hub.docker.com/_/python/tags?page=1&name=3.11-slim-bullseye
FROM docker.io/python:3.11-slim-bullseye

LABEL maintainer="jingnanl@amazon.com"
#增加语言utf-8
ENV LANG=zh_CN.UTF-8
ENV LC_CTYPE=zh_CN.UTF-8
ENV LC_ALL=C
ENV PYTHONPATH=/data/InStock
EXPOSE 9988

RUN echo  "[global]\n\
index-url = https://pypi.org/simple\n\
trusted-host = pypi.org" > /etc/pip.conf && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    apt-get update && \
    apt-get install -y cron gcc make python3-dev default-libmysqlclient-dev build-essential pkg-config curl && \
    pip install supervisor && \
    pip install mysqlclient && \
    pip install requests && \
    pip install arrow && \
    pip install numpy && \
    pip install SQLAlchemy && \
    pip install PyMySQL && \
    pip install Logbook && \
    pip install python_dateutil && \
    pip install py_mini_racer && \
    pip install tqdm && \
    pip install beautifulsoup4 && \
    pip install bokeh && \
    pip install pandas && \
    pip install tornado && \
    pip install mini-racer && \
    pip install easytrader

RUN mkdir ta-lib && \
    curl -SL $(curl -s https://api.github.com/repos/ta-lib/ta-lib/releases/latest | grep browser_download_url | cut -d'"' -f4 |grep -E 'tar.gz$') | tar -xz --strip-components=1 -C ./ta-lib && \
    cd ta-lib/ && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    pip install TA-Lib && \
    rm -rf ta-lib*

RUN apt-get --purge remove -y gcc make python3-dev default-libmysqlclient-dev curl && \
    rm -rf /root/.cache/* && rm -rf /var/lib/apt/lists/* && apt-get clean && apt-get autoclean && apt-get autoremove -y

WORKDIR /data
#InStock软件
COPY stock /data/InStock
COPY cron/cron.hourly /etc/cron.hourly
COPY cron/cron.workdayly /etc/cron.workdayly
COPY cron/cron.monthly /etc/cron.monthly

#add cron sesrvice.
#任务调度
RUN chmod 755 /data/InStock/instock/bin/run_*.sh && \
    chmod 755 /etc/cron.hourly/* && chmod 755 /etc/cron.workdayly/* && chmod 755 /etc/cron.monthly/* && \
    echo "SHELL=/bin/sh \n\
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin \n\
# min hour day month weekday command \n\
*/30 9,10,11,13,14,15 * * 1-5 /bin/run-parts /etc/cron.hourly \n\
30 17 * * 1-5 /bin/run-parts /etc/cron.workdayly \n\
30 10 * * 3,6 /bin/run-parts /etc/cron.monthly \n" > /var/spool/cron/crontabs/root && \
    chmod 600 /var/spool/cron/crontabs/root

ENTRYPOINT ["supervisord","-n","-c","/data/InStock/supervisor/supervisord.conf"]