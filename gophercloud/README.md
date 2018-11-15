# Gophercloud：OpenStack的Go SDK
[![Build Status](https://travis-ci.org/gophercloud/gophercloud.svg?branch=master)](https://travis-ci.org/gophercloud/gophercloud)
[![Coverage Status](https://coveralls.io/repos/github/gophercloud/gophercloud/badge.svg?branch=master)](https://coveralls.io/github/gophercloud/gophercloud?branch=master)

Gophercloud是OpenStack的GoSDK。

## 友情链接

* [参考文档](http://godoc.org/github.com/gophercloud/gophercloud)
* [高效的Go](https://golang.org/doc/effective_go.html)

## 如何安装

安装之前，需要确保你的[GOPATH环境变量](https://golang.org/doc/code.html#GOPATH)指向你想要安装Gophercloud的目录。

```bash
mkdir $HOME/go
export GOPATH=$HOME/go
```

为了使自己不受依赖变化的影响，我们强烈建议你给项目选择一个[依赖管理解决方案](https://github.com/golang/go/wiki/PackageManagementTools)，例如[godep](https://github.com/tools/godep)。一旦设置好这些，你就可以安装Gophercloud依赖，如下所示：

```bash
go get github.com/gophercloud/gophercloud

# Edit your code to import relevant packages from "github.com/gophercloud/gophercloud"

godep save ./...
```

这将会把你所需的全部源文件安装到`Godeps/_workspace`目录中，当你使用`godep go`命令时，你自己的源文件中可以引用`Godeps/_workspace`目录中的源文件。

## 入门指南

### 证书

因为你将会使用一些API，因此你需要获取你的OpenStack证书，并把证书存储在环境变量或本地Go文件中。推荐使用第一种方式，因为这样可以把证书信息从源码中分离出来，让你在没有任何安全风险的情况下，把源码push到你的版本控制系统中。

你需要获取以下信息：

* 用户名（username）
* 密码（password）
* 有效的Keystone identity URL

For users that have the OpenStack dashboard installed, there's a shortcut. If
you visit the `project/access_and_security` path in Horizon and click on the
"Download OpenStack RC File" button at the top right hand corner, you will
download a bash file that exports all of your access details to environment
variables. To execute the file, run `source admin-openrc.sh` and you will be
prompted for your password.

### 认证

一旦你有了访问证书，你就可以开始把它们插入到Gophercloud中。下一步是身份认证，这是由一个基础的"Provider"结构体（struct）来处理的。要获取一个Provider，你可以显示地传递你的证书，或者告诉Gophercloud使用环境变量。

```go
import (
  "github.com/gophercloud/gophercloud"
  "github.com/gophercloud/gophercloud/openstack"
  "github.com/gophercloud/gophercloud/openstack/utils"
)

// Option 1: Pass in the values yourself
opts := gophercloud.AuthOptions{
  IdentityEndpoint: "https://openstack.example.com:5000/v2.0",
  Username: "{username}",
  Password: "{password}",
}

// Option 2: Use a utility function to retrieve all your environment variables
opts, err := openstack.AuthOptionsFromEnv()
```

Once you have the `opts` variable, you can pass it in and get back a
`ProviderClient` struct:

```go
provider, err := openstack.AuthenticatedClient(opts)
```

The `ProviderClient` is the top-level client that all of your OpenStack services
derive from. The provider contains all of the authentication details that allow
your Go code to access the API - such as the base URL and token ID.

### 提供服务器

Once we have a base Provider, we inject it as a dependency into each OpenStack
service. In order to work with the Compute API, we need a Compute service
client; which can be created like so:

```go
client, err := openstack.NewComputeV2(provider, gophercloud.EndpointOpts{
  Region: os.Getenv("OS_REGION_NAME"),
})
```

We then use this `client` for any Compute API operation we want. In our case,
we want to provision a new server - so we invoke the `Create` method and pass
in the flavor ID (hardware specification) and image ID (operating system) we're
interested in:

```go
import "github.com/gophercloud/gophercloud/openstack/compute/v2/servers"

server, err := servers.Create(client, servers.CreateOpts{
  Name:      "My new server!",
  FlavorRef: "flavor_id",
  ImageRef:  "image_id",
}).Extract()
```

The above code sample creates a new server with the parameters, and embodies the
new resource in the `server` variable (a
[`servers.Server`](http://godoc.org/github.com/gophercloud/gophercloud) struct).

## 高级用法

Have a look at the [FAQ](./docs/FAQ.md) for some tips on customizing the way Gophercloud works.

## 向后兼容保证

None. Vendor it and write tests covering the parts you use.

## 贡献

请参阅[贡献指南](./.github/CONTRIBUTING.md)。

## 帮助及反馈

If you're struggling with something or have spotted a potential bug, feel free
to submit an issue to our [bug tracker](https://github.com/gophercloud/gophercloud/issues).

## 鸣谢

We'd like to extend special thanks and appreciation to the following:

### OpenLab

<a href="http://openlabtesting.org/"><img src="./docs/assets/openlab.png" width="600px"></a>

OpenLab is providing a full CI environment to test each PR and merge for a variety of OpenStack releases.

### VEXXHOST

<a href="https://vexxhost.com/"><img src="./docs/assets/vexxhost.png" width="600px"></a>

VEXXHOST is providing their services to assist with the development and testing of Gophercloud.
