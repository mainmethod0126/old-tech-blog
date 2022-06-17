# Logstash 사용법

## config/logstash.yml

### pipeline.workers

만약 더 많은 cpu코어를 logstash 작업에 사용할 거라면 해당 옵션의 값을 늘리면 됩니다.

### pipeline.output.woerkers

output 별로 작업을 담당할 worker 수 지정, 늘리면 더 많은 병렬처리가 가능합니다.

### pipeline.batch.size

들어오는 event data를 몇개씩 묶어서 배치를 만들건지 지정합니다. (기본 125개)

### pipeline.batch.delay

해당 값의 단위는 밀리초 이며
내가 만든 배치를 몇 초마다 목적지(output)으로 던질지를 정합니다.
> **logstash 스루풋(throughput) 높이는 법**
> batch.size 를 늘리고 batch.delay를 줄이면 출력 스루풋을 높일 수 있습니다.
> 다만, 이렇게 하려면 logstash가 사용하는 heap memory 량을 많이 늘려야 할 수 있습니다.
> **스루풋 이란?**
> 스루풋 또는 처리율은 통신에서 네트워크 상의 어떤 노드나 터미널로부터 또 다른 터미널로 전달되는 단위 시간당 디지털 데이터 전송으로 처리하는 양을 말한다.

### config.reload.automatic

logstash는 설치하고 바로 실행하면 오류가납니다.
기본적으로 입력과 아웃에 대한 정의가 되어있어야 오류가 발생하지 않습니다.
이 입력, 아웃에 대한 정의 변경을 감지하는 부분이 config.reload.automatic 입니다.

### config.reload.interval

입력, 아웃에 대한 정의변경을 얼마 주기로 감지할지에대한 설정입니다.

## modules

프리셋 지정

## config/pipelines.yml

## Logstash pipeline 지정 옵션

logstash가 사용할 파이프라인은 배열로 등록할 수 있습니다.
즉 멀티가 됩니다.

```text
# Example of two pipelines:

# - pipeline.id: test
#   pipeline.workers: 1
#   pipeline.batch.size: 1
#   config.string: "input { generator {} } filter { sleep { time => 1 } } output { stdout { codec => dots } }"
# - pipeline.id: another_test
#   queue.type: persisted
#   path.config: "/tmp/logstash/*.config"

# 위 Example을 참고하여 아래와 같이 작성할 수 있습니다.
 - pipeline.id: my-pipeline
   path.config: "/home/softcamp/logstash/pipelines/my-pipeline.config"
```

```text
// my-pipeline.config
input {
 tcp { 
    port => 9900 // 입력을 tcp 로 받습니다
 }
}

filter {
    grok { // grok filter를 이용하여 입력받은 message 에서 "Hello" 이후의 텍스트를 name 이란 변수에 저장합니다
        match => { "message" => "Hello %{WORD:name}" }
    }
}

output {
 stdout {
    codec => rubydebug
 }
}
```

### -e

실행인자로 input,output 를 지정할 수 있습니다.
아래와 같이 옵션 -e에 인자를 넣으면 실행이 됩니다.

```bash
logstash -e 'input { stdin {} } output { stdout {} }'
```

### -f

미리 설정해둔 iuput,output 파일을 선택하여 실행할 수 있습니다.

설정 파일은 아래와 같이 작성할 수 있으며,

```text
// simple.config
input {
    stdin { }
}

output {
    stdout {
        codec => rubydebug
    }
}

```

아래와 같이 옵션 -f에 작성한 simple.config 를 인자로 넣으면 실행이 됩니다.

```bash
logstash -f simple.config
```

