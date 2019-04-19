FROM centos
RUN yum install -y epel-release
RUN yum install -y python-pip
RUN pip install flask
COPY hello.py /hello.py
ENTRYPOINT ["python", "hello.py"]
EXPOSE 5000/tcp
