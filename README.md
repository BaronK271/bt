[![gitter](https://img.shields.io/gitter/room/leopotam/bt.svg)](https://gitter.im/leopotam/ecs)
[![discord](https://img.shields.io/discord/404358247621853185.svg?label=discord)](https://discord.gg/5GZVde6)
[![license](https://img.shields.io/github/license/Leopotam/bt.svg)](https://github.com/Leopotam/bt/blob/develop/LICENSE)
# LeoBT - Simple lightweight C# Behaviour Tree framework
Code only, small codebase, no dependencies on any game engine - main goals of this framework.

> Tested on unity 2019.1 (not dependent on it) and contains assembly definition for compiling to separate assembly file for performance reason.

> **Its early work-in-progress stage, not recommended to use in real projects, any api / behaviour can change later.**

> **Important!** Dont forget to use `DEBUG` builds for development and `RELEASE` builds in production: all internal error checks / exception throwing works only in `DEBUG` builds and eleminated for performance reasons in `RELEASE`.

# Installation

## As unity module
This repository can be installed as unity module directly from git url. In this way new line should be added to `Packages/manifest.json`:
```
"com.leopotam.ecs": "https://github.com/Leopotam/bt.git",
```
By default last released version will be used. If you need trunk / developing version then `develop` name of branch should be added after hash:
```
"com.leopotam.ecs": "https://github.com/Leopotam/bt.git#develop",
```

## As source
If you can't / don't want to use unity modules, code can be downloaded as sources archive of required release from [Releases page](`https://github.com/Leopotam/bt/releases`).

# Example
```csharp
class BtStore {
    public int State1;
    public int State2;
    public int State3;
}

class BtTest : MonoBehaviour {
    void Start () {
        var bt = new Bt<BtStore> (new BtStore ());
        // Sequence test.
        bt.Root
            .Then (OnNode1)
            .Then (OnNode2)
            .Then (store => {
                Debug.Log ("lambda node - ok");
                return BtResult.Success;
            })

            // Nested BehaviourTreeSequence test.
            .Seq ()
            .Then (store => {
                Debug.Log ("internal_sequence1 - ok");
                return BtResult.Success;
            })
            .Then (store => {
                Debug.Log ("internal_sequence2 - ok");
                return BtResult.Success;
            });

        // Parallel test.
        bt.Root
            .Parallel ()
            .Then (store => {
                Debug.Log ("parallel_node1 - ok");
                return BtResult.Success;
            })
            .Then (store => {
                store.State2++;
                if (store.State2 < 2) {
                    Debug.Log ("parallel_node2 - pending");
                    return BtResult.Pending;
                }
                Debug.Log ("parallel_node2 - ok");
                return BtResult.Success;
            })

            // Condition test.
            .When (store => {
                store.State3++;
                return store.State3 >= 2 ? BtResult.Success : BtResult.Fail;
            })
            .Then (store => {
                Debug.Log ("wow, pending counter >= 2!");
                return BtResult.Success;
            });

        // Select test.
        bt.Root
            .Select ()
            .Then (store => {
                Debug.Log ("will be processed");
                return BtResult.Fail;
            })
            .Then (store => {
                Debug.Log ("will be processed too");
                return BtResult.Fail;
            })
            .Then (store => {
                Debug.Log ("will be processed and stopped on it");
                return BtResult.Success;
            })
            .Then (store => {
                Debug.Log ("will not be processed because previous node returned positive result");
                return BtResult.Fail;
            });

        // Bt ready to run.
        BtResult res;
        do {
            res = bt.Run ();
            Debug.Log (">>> " + res);
        } while (res == BtResult.Pending);
    }

    BtResult OnNode1 (BtStore store) {
        store.State1++;
        if (store.State1 < 2) {
            Debug.Log ("node1 - pending");
            return BtResult.Pending;
        }
        Debug.Log ("node1 - ok");
        return BtResult.Success;
    }

    BtResult OnNode2 (BtStore store) {
        Debug.Log ("node2 - ok");
        return BtResult.Success;
    }
}
```
Output:
```
node1 - pending
>>> Pending
node1 - ok
node2 - ok
lambda node - ok
internal_sequence1 - ok
internal_sequence2 - ok
parallel_node1 - ok
parallel_node2 - pending
>>> Pending
parallel_node1 - ok
parallel_node2 - ok
wow, pending counter >= 2!
will be processed
will be processed too
will be processed and stopped on it
>>> Success
```
# License
The software released under the terms of the [MIT license](./LICENSE). Enjoy.

# Donate
Its free opensource software, but you can buy me a coffee:

<a href="https://www.buymeacoffee.com/leopotam" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/yellow_img.png" alt="Buy Me A Coffee" style="height: auto !important;width: auto !important;" ></a>