#### SIG TFX-Addons
# Project Proposal

---

**Your name:** Chansung Park

**Your email:** deep.diver.csp@gmail.com

**Your company/organization:** Individual ([ML GDE](https://developers.google.com/community/experts/directory/profile/profile-chansung-park))

**Project name:** HuggingFace Model Pusher

## Project Description
HuggingFace Model Pusher(`HFModelPusher`) pushes blessed model to the [HuggingFace Model Hub](https://huggingface.co/models).

## Project Category
Component

## Project Use-Case(s)
The HuggingFace Model Hub lets us have [Git-LFS](https://git-lfs.github.com) enabled repositories in public and private modes. Supported models hosted on the HuggingFace Model Hub can be directly loaded/used with APIs provided by [transformers](https://huggingface.co/docs/transformers/index) package. However, it is not limited. We can host arbitrary types of models too. 

HuggingFace Model Hub is easy to manage model versions, especially for those familiar with Git.

## Project Implementation
HFModelPusher is a class-based TFX component, and it inherits from TFX standard `Pusher` component.

It takes the following inputs:
```
HFModelPusher(
    username: str,
    huggingface_access_token: str,
    repo_name: Optional[str],    
    model: Optional[types.Channel] = None,
    model_blessing: Optional[types.Channel] = None,    
)
```
- `username` : username of the HuggingFace user (can be an individual user or an organization)
- `hf_access_token` : access token value of the HuggingFace user. 
- `repo_name` : the repository name to push the current version of the model to. The default value is same as the TFX pipeline name
- `model` : the model artifact from the upstream TFX component such as `Trainer`
- `model_blessing` : the blessing artifact from the upstream TFX component such as `Evaluator`

It gives the follwing outputs:
- `pushed` : integer value to denote if the model is pushed or not. This is set to 0 when the input model is not blessed, and it is set to 1 when the model is successfully pushed
- `pushed_version` : string value to indicate the current model version. This is decided by `time.time()` Python built-in function
- `repo_id` : repository ID where the model is pushed to. This follows the format of f"{username}/{repo_name}"
- `branch` : branch name where the model is pushed to. The branch name is automatically assigned to the same value of  `pushed_version`
- `commit_id` : the id from the commit history (branch name could be sufficient to retreive a certain version of the model)
- `repo_url` : repository URL. It is something like f"https://huggingface.co/{repo_id}/{branch}"

The behaviour of the component:
1. It pushes the model when the `model` is blessed, or it pushes the `model` when the `model_blessing` parameter is set to `None`. This behaviour inherits from the standard `Pusher` component
2. Creates HuggingFace Hub Repository object using the `huggingface-hub` package. It will clone one if there is already an existing repository
3. Checks out a new branch with the name as `pushed_version`. Since the model is pushed for experimental purpose, it would be good to track the versions of the model within separate branches (When the model is ready to be open to public, one can manually merge the right version(branch) into the main branch)
4. Copy all the model related files into a temporary directory in a local file system. All the model related files produced by the upstream component such as `Trainer`. They could be stored in GCS bucket, so `tf.io.gfile` module is a good choice since it handles files in location agnostic manner (GCS or local)
5. Add & commit the current status
6. Pushes the commit to the remote HuggingFace Model Repository


## Project Dependencies
- [tfx](https://pypi.org/project/tfx/)
- [huggingface-hub](https://pypi.org/project/huggingface-hub/)

## Project Team
- Chansung Park, @deep-diver, deep.diver.csp@gmail.com
- Sayak Paul, @sayakpaul, spsayakpaul@gmail.com

# Note
Please be aware of the processes and requirements which are outlined here:

* [SIG-TFX-Addons](https://github.com/tensorflow/tfx-addons)
* [Contributing Guidelines](https://github.com/tensorflow/tfx-addons/blob/main/CONTRIBUTING.md)
* [TensorFlow Code of Conduct](https://github.com/tensorflow/tfx-addons/blob/main/CODE_OF_CONDUCT.md)