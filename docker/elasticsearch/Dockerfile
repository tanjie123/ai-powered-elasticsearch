FROM elasticsearch:7.9.3

RUN bin/elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
RUN bin/elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.9.3/elasticsearch-analysis-pinyin-7.9.3.zip
RUN bin/elasticsearch-plugin install -b https://github.com/tanjie123/elasticsearch-analysis-dynamic-synonym/releases/download/7.9.3/elasticsearch-analysis-dynamic-synonym-7.9.3.zip
RUN bin/elasticsearch-plugin install -b https://github.com/o19s/elasticsearch-learning-to-rank/releases/download/v1.5.3-es7.9.3/ltr-plugin-v1.5.3-es7.9.3.zip

COPY analysis/synonyms.txt /usr/share/elasticsearch/config/analysis/
COPY analysis-ik/ /usr/share/elasticsearch/config/analysis-ik/