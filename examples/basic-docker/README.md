# qri-dokku

A dockerfile and documentation for running a [qri](https://qri.io) instance as a [dokku](http://dokku.viewdocs.io/dokku/) app.

## Background

The original goal was to provide a way to automate the update of a qri dataset.  Using qri on the CLI or desktop generally means running it on a personal computer.  Automating a dataset update, say appending a row every hour, requires the machine to always be on/available for an automation script to interact with.

Because qri comes with a built-in http server we can simply run it in the cloud and interact with it via http. Using dokku, we can eliminate some of the headaches of setting up a containerized cloud service (dokku will handle the domain names, volumes, etc, and can automatically swap out containers when new code is pushed) 

For an example of an automated dataset update script that uses this dokku-ized qri, see [`brooklyn-weather-bot`](https://github.com/chriswhong/qri-weather-bot)

## Setup

Running qri on dokku is as simple as creating a new dokku app, mapping a volume so that the app can persist data, setting up a qri user, and adding a domain name.

1. Create a new dokku app

SSH into your dokku server and create a new dokku app.  I have been choosing names prefixed with `qri-` that can also serve as the app's subdomain on the primary domain I use on my dokku server.  

`dokku apps:create qri-test`

2. Mount storage

Mount storage on the VPS to `/root/.qri`.  This will contain configuration files, logs, and all data used by the qri instance.  In this example I have a directory `/root/qri-profiles` which contains a directory named for each `qri-dokku` instance.

`dokku storage:mount qri-test /root/qri-profiles/qri-test:/root/.qri`

3. Push this repo

dokku will pick up the `Dockerfile`, build it, and run the container.

`git remote add dokku dokku@{host}:{appname}`
`git push dokku master`

4. Sign up for a qri.cloud account

The dockerfile runs `qri setup -a`, which initializes the qri repo with a local username, but does not register an account with qri cloud.

If we want to be able to publish to cloud, we must complete a signup by executing a `qri registry` command inside the running container:

`dokku run qri-test qri registry signup --username qri-dokku-test --email chris+qri-dokku-test@qri.io`

5. Add a domain name

`dokku domains:add qri-test qri-test.mydomain.com`


6. Map ports

The qri http server is running on port `2503`.  We must map port 2503 for this app's domain name to 2503 in the container:

`dokku proxy:ports-add qri-test http:2503:2503`

## Using the cloud qri instance from other environments

Here are some snippets of node.js code that interact with a qri service:

#### Getting a dataset body

```js
const downloadLatest = async ({ username, dsname, filePath, qriHost }) => {
  return new Promise(async (resolve, reject) => {
    console.log('download')
    const response = await fetch(`${qriHost}/body/${username}/${dsname}?download=true`)
    if (!response.ok) {
      fs.createWriteStream(filePath).write(headerRow)
    } else {
      await streamPipeline(response.body, fs.createWriteStream(filePath))
    }
    resolve()
  })
}
```

#### Commit a new version of a dataset (from a CSV)

```js
// create/update the qri dataset using a cloud-hosted qri instance
let formData = new FormData()
formData.append('body', fs.createReadStream(filePath))
formData.append('peername', username)
formData.append('name', dsname)

// save
console.log(`saving qri dataset ${username}/${dsname}...`)
await fetch(`${qriHost}/save`, {
  method: 'POST',
  body: formData
})
  .then(res => res.json())
  .then(json => console.log(json))
```

#### Publishing a dataset to qri.cloud

```js
await fetch(`${qriHost}/push/${username}/${dsname}`, {
  method: 'POST'
})
```