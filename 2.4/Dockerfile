# Blatantly ripped off from https://github.com/docker-library/elasticsearch/blob/1a539d229073ccde66f94487889d17d939f9689c/2.4/Dockerfile
# Because we need a JDK, not JRE
FROM openjdk:8-jdk

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true

RUN set -ex; \
# https://artifacts.elastic.co/GPG-KEY-elasticsearch
	key='46095ACC8548582C1A2699A9D27D666CD88E42B4'; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	gpg --export "$key" > /etc/apt/trusted.gpg.d/elastic.gpg; \
	rm -r "$GNUPGHOME"; \
	apt-key list

# https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html
# https://www.elastic.co/guide/en/elasticsearch/reference/5.0/deb.html
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends apt-transport-https && rm -rf /var/lib/apt/lists/* \
	&& echo 'deb http://packages.elasticsearch.org/elasticsearch/2.x/debian stable main' > /etc/apt/sources.list.d/elasticsearch.list

ENV ELASTICSEARCH_VERSION 2.4.3
ENV ELASTICSEARCH_DEB_VERSION 2.4.3

RUN set -x \
	\
# don't allow the package to install its sysctl file (causes the install to fail)
# Failed to write '262144' to '/proc/sys/vm/max_map_count': Read-only file system
	&& dpkg-divert --rename /usr/lib/sysctl.d/elasticsearch.conf \
	\
	&& apt-get update \
	&& apt-get install -y --no-install-recommends "elasticsearch=$ELASTICSEARCH_DEB_VERSION" \
	&& rm -rf /var/lib/apt/lists/*

ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

RUN set -ex \
	&& for path in \
		./data \
		./logs \
		./config \
		./config/scripts \
	; do \
		mkdir -p "$path"; \
		chown -R elasticsearch:elasticsearch "$path"; \
	done

COPY config ./config

VOLUME /usr/share/elasticsearch/data

COPY docker-entrypoint.sh /

ADD mecab-java-0.996 /tmp/libmecab

# Install mecab-ko, mecab-ko-dic, libMeCab.so
ENV MECAB_VER mecab-0.996-ko-0.9.2
ENV MECAB_DIC_VER mecab-ko-dic-2.0.1-20150920
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends build-essential autotools-dev automake ; \
	wget -O /tmp/mecab-ko.tar.gz https://bitbucket.org/eunjeon/mecab-ko/downloads/$MECAB_VER.tar.gz \
	&& mkdir /tmp/mecab-ko && tar -xzf /tmp/mecab-ko.tar.gz -C /tmp/mecab-ko && cd /tmp/mecab-ko/$MECAB_VER \
	&& ./configure && make && make check && make install \
	&& ldconfig; \
	wget -O /tmp/mecab-ko-dic.tar.gz https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/$MECAB_DIC_VER.tar.gz \
	&& mkdir /tmp/mecab-ko-dic && tar -xzf /tmp/mecab-ko-dic.tar.gz -C /tmp/mecab-ko-dic && cd /tmp/mecab-ko-dic/$MECAB_DIC_VER \
	&& ./autogen.sh && ./configure && make && make install; \
	cd /tmp/libmecab && ls -al && make && cp libMeCab.so /usr/local/lib; \
	apt-get remove -y build-essential autotools-dev automake \
	&& rm -rf /var/lib/apt/lists/* ;\
	rm -rf /tmp/mecab-ko.tar.gz /tmp/mecab-ko /tmp/mecab-ko-dic.tar.gz /tmp/mecab-ko-dic /tmp/libmecab

# Install elasticsearch plugin
WORKDIR /usr/share/elasticsearch 
RUN ./bin/plugin install https://bitbucket.org/eunjeon/mecab-ko-lucene-analyzer/downloads/elasticsearch-analysis-mecab-ko-2.4.3.0.zip

ENV LD_LIBRARY_PATH=/usr/local/lib

EXPOSE 9200 9300
ENTRYPOINT ["/docker-entrypoint.sh"]
# https://groups.google.com/forum/?hl=ko#!topic/eunjeon/i9vqL3Uw77s
CMD ["elasticsearch", "-Des.security.manager.enabled=false"]
