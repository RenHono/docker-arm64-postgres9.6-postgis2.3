FROM --platform=linux/arm64 arm64v8/postgres:9.6.23-stretch

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb http://mirrors.163.com/debian/ stretch main non-free contrib" >/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian/ stretch-backports main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ stretch main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ stretch-updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ stretch-backports main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib" >>/etc/apt/sources.list


RUN echo " " > /etc/apt/sources.list.d/pgdg.list

RUN apt-get update && apt-get install -y \
	apt-transport-https 

RUN echo "deb https://mirrors.huaweicloud.com/postgresql/repos/apt stretch-pgdg main" > /etc/apt/sources.list.d/pgdg.list

RUN	apt-get update && apt-get install -y postgresql-9.6-postgis-2.3 \
	postgresql-9.6-mysql-fdw \
	postgresql-plpython3-9.6 \
	postgresql-server-dev-9.6 \	
	make \
	gcc \
	unzip \
	libaio1 \
	libaio-dev


COPY ext/pg_cron /opt/pg_cron
COPY ext/oracle_fdw-ORACLE_FDW_2_4_0 /opt/oracle_fdw
RUN mkdir /opt/oracle
ADD ext/oracle/* /opt/oracle/

WORKDIR /opt/oracle

RUN pwd
RUN ls -la

WORKDIR /opt/pg_cron
RUN ls -la

RUN export PATH=/usr/lib/postgresql/9.6/bin:$PATH
RUN make && PATH=$PATH make install
#RUN make && make install

RUN chmod -R 777 /var/lib/postgresql/data

RUN echo "shared_preload_libraries = 'pg_cron'" >> /var/lib/postgresql/data/postgresql.conf
RUN echo "cron.database_name='postgres'" >> /var/lib/postgresql/data/postgresql.conf

WORKDIR /usr/share/postgresql/9.6/extension
RUN cp pg_cron--1.0.sql pg_cron--1.5.sql

WORKDIR /opt/oracle
RUN unzip instantclient-basic-linux.arm64-19.10.0.0.0dbru.zip 
RUN unzip instantclient-sdk-linux.arm64-19.10.0.0.0dbru.zip
RUN unzip instantclient-sqlplus-linux.arm64-19.10.0.0.0dbru.zip
RUN mv instantclient_19_10 instantclient

RUN ls -al
RUN ls -lah instantclient

#WORKDIR /opt/oracle_fdw

#RUN export ORACLE_HOME=/opt/oracle/instantclient
#RUN export OCI_LIB_DIR=$ORACLE_HOME
#RUN export OCI_INC_DIR=$ORACLE_HOME/sdk/include
#RUN export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH

#RUN make && make install

WORKDIR /etc/ld.so.conf.d

RUN echo "/opt/oracle/instantclient" > oracle-arm64.conf
RUN ldconfig 

WORKDIR /opt/oracle_fdw

ENV ORACLE_HOME=/opt/oracle/instantclient
ENV export OCI_LIB_DIR=$ORACLE_HOME
ENV export OCI_INC_DIR=$ORACLE_HOME/sdk/include
ENV export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH

RUN env
RUN echo $ORACLE_HOME
RUN make && make install


WORKDIR /var/lib/postgresql/data

#RUN echo "shared_preload_libraries = 'pg_cron'" >> /var/lib/postgresql/data/postgresql.conf
#RUN echo "cron.database_name='postgres'" >> /var/lib/postgresql/data/postgresql.conf


VOLUME /var/lib/postgresql/data

EXPOSE 5432

CMD ["postgres"]

