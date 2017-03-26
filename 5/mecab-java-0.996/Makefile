# Src: https://drive.google.com/drive/u/0/folders/0B4y35FiV1wh7fjQ5SkJETEJEYzlqcUY4WUlpZmR4dDlJMWI5ZUlXN2xZN2s2b0pqT3hMbTQ
# Ref: https://bitbucket.org/eunjeon/mecab-ko-lucene-analyzer/raw/master/elasticsearch-analysis-mecab-ko/
TARGET=MeCab
JAVAC=javac
JAVA=java
JAR=jar
CXX=c++
INCLUDE=/usr/lib/jvm/java-8-openjdk-amd64/include

PACKAGE=org/chasen/mecab

LIBS=`mecab-config --libs`
INC=`mecab-config --cflags` -I$(INCLUDE) -I$(INCLUDE)/linux

all:
	$(CXX) -O1 -c -fpic $(TARGET)_wrap.cxx  $(INC)
	$(CXX) -shared  $(TARGET)_wrap.o -o lib$(TARGET).so $(LIBS)
	$(JAVAC) $(PACKAGE)/*.java
	$(JAVAC) -cp . test.java
	$(JAR) cfv $(TARGET).jar $(PACKAGE)/*.class

test:
	env LD_LIBRARY_PATH=. $(JAVA) test

clean:
	rm -fr *.jar *.o *.so *.class $(PACKAGE)/*.class
	
cleanall:
	rm -fr $(TARGET).java *.cxx
