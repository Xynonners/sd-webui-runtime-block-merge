# auto-MBW-rt "runtime enhancement" by 6DammK9 (1.7.0 Tested)

Minor fix and enhancements on top of [Xynonners' Fork](https://github.com/Xynonners/sd-webui-runtime-block-merge). Development along with [my fork of auto-MBW-rt](https://github.com/6DammK9/auto-MBW-rt/tree/master).

## Installation guide

- "Install as usual" (Install these extensions via "Extensions" > "Install from URL", or just clone it in `SD_DIR/extension`). There is no change on AI related stuffs. This is SE focused.

## Major change

- **Add SDXL support.** See next session.

- Fix the awful inconsistancy between UI, runtime, and AutoMBW.

```py
sl_ALL_nat = [*sl_INPUT, *sl_MID, sl_OUT, *sl_OUTPUT, sl_TIME_EMBED]
sl_ALL = [*sl_INPUT, *sl_MID, *sl_OUTPUT, sl_TIME_EMBED, sl_OUT]

# Should be sl_ALL = [*sl_INPUT, *sl_MID, *sl_OUTPUT, sl_OUT, sl_TIME_EMBED]
```

- Add the logger, and probably reformat the log (if possible)

- Add `.gitignore`.

- Merge the info to the `__metadata__` inside the model: [How the original Checkpoint Merger does](https://github.com/AUTOMATIC1111/stable-diffusion-webui/blob/master/modules/extras.py#L257)

```py
safetensors.torch.save_file(combined_state_dict, save_checkpoint_path, metadata=metadata if len(metadata)>0 else None)
```

## Notes on merging SDXL models

- Due to ~~messy codebase~~ backward compatibility, it is adviced to restart WebUI if your next merge is in different SD version, and make sure you have booted WebUI with a SDXL model loaded. Otherwise you will see loads of `Missing key(s) in state_dict:`. **Therefore you need 3 SDXLs in total to operate.**

- This extension supports most layers, but `label_emb` will be untouched. ~~I don't want to add another "slider" to make things overcomplicated.~~

- **IN09-IN11 and OUT09-OUT11 will be ignored for merging SDXL.** UI / algorithm mappings will be untouched. For AutoMBW, it will still includes all 27 parameters to optimize, but you can expect they are *irrelevant* to the merge, and the algorithm (e.g. `BayesianOptimization`) will take care of it. ~~noisy environment is common in AI/ML, so no worry.~~ 

```py
# To support SDXL, we have found that IN09-IN11 and OUT09-OUT11 are not exist, then we can skip them.
# Meanwhile label_emb is SDXL exclusive, we need to ignore it.
# Before: current_block_index++
```

## Read metadata inside the model

- Read by [safetensors_util](https://github.com/by321/safetensors_util), or read the `extras.py`.

- If success, you will see a JSON string:

```json
{
    "__metadata__": {
        "sd_mbwrt_receipe": {
            "type": "auto-mbw-rt",
            "modelA": "_03a-mzpikas_tmnd_enhanced-sd-v1-4.safetensors",
            "modelB": "_04a-dreamshaper_8-sd-v1-4.safetensors",
            "modelO": "03b-verify",
            "position_id_fix": "Keep Original",
            "output_mode": "Max Precision",
            "mbwrt_weights": "0.3,0.8,1,0.5,1,0,0.7,0.9,0.5,0.9,0.2,0.2,0.6,0,0.3,0.3,1,1,0.8,0.2,0.7,0,1,0.3,0.2,0.9,0.2",
            "mbwrt_weights_seq": "[*sl_INPUT, *sl_MID, *sl_OUTPUT, sl_OUT, sl_TIME_EMBED]"
        },
        "format": "pt"
    }
}
```

## Some observation.

- The very bottom of "merging algorithm" is `torch.lerp`. Therefore efficiency is good, but it won't support add-diff.

## This is part of my research.

- Just a hobby. [If you are feared by tuning for numbers, try "averaging" by simply 0.5, 0.33, 0.25... for 20 models. It works.](https://github.com/6DammK9/nai-anime-pure-negative-prompt/tree/main/ch05).

------

# sd-webui-runtime-block-merge

Runtime version of Merge Block Weighted - GUI, let's you preview the effect before you merge.
Credits to original author and repo:
https://github.com/bbc-mc/sdweb-merge-block-weighted-gui
