This experiment is generated using [MLCommons CM](https://github.com/mlcommons/ck)
## CM Run Command
```
cm run script \
	--tags=generate-run-cmds,inference,_submission \
	--model=retinanet \
	--implementation=cpp \
	--device=cpu \
	--backend=onnxruntime \
	--scenario=Offline \
	--category=edge \
	--division=closed \
	--quiet \
	--adr.compiler.tags=gcc \
	--execution-mode=valid \
	--skip_submission_generation=yes \
	--power=yes \
	--adr.mlperf-power-client.power_server=192.168.0.15 \
	--adr.mlperf-power-client.port=4950 \
	--results_dir=/home/arjun/results_dir
```