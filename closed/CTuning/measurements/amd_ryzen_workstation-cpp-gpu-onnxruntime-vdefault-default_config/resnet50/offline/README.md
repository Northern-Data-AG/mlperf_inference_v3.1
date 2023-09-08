This experiment is generated using [MLCommons CM](https://github.com/mlcommons/ck)
## CM Run Command
```
cm run script \
	--tags=generate-run-cmds,inference,_submission \
	--scenario=Offline \
	--model=resnet50 \
	--implementation=cpp \
	--device=cuda \
	--backend=onnxruntime \
	--category=edge \
	--division=closed \
	--quiet \
	--adr.compiler.tags=gcc \
	--execution-mode=valid \
	--skip_submission_generation=yes \
	--env.pCM_MLPERF_LOADGEN_MAX_BATCHSIZE=1 \
	--results_dir=/home/arjun/results_dir
```