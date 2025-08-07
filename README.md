[![Automatic version updates](https://github.com/zopencommunity/grpcport/actions/workflows/bump.yml/badge.svg)](https://github.com/ZOSOpenTools/grpcport/actions/workflows/bump.yml)

# grpc

gRPC is a remote procedure call framework

# Installation and Usage

Use the zopen package manager ([QuickStart Guide](https://zopen.community/#/Guides/QuickStart)) to install:
```bash
zopen install grpc
```

# Building from Source

1. Clone the repository:
```bash
git clone https://github.com/zopencommunity/grpcport.git
cd grpcport
```
2. Build using zopen:
```bash
zopen build -vv
```

See the [zopen porting guide](https://zopen.community/#/Guides/Porting) for more details.

# Documentation

## Patch Explanations

### Thread Local

grpc uses tls objects waaaaaay too much. As a result, I had to litter their codebase with a bunch of conditionally compiled zoslib ''__tlssim'' workarounds. Blame grpc :)

### Event polling engine

Disabled linux-based polling (``epoll``) since some linux polling functionality is missing on z/OS. Igor brought to my attention there's actually an environment variable that you can change that controls the polling strategy. I directly disabled the relevant macro in the code, but the environment approach is probably better.

### Aligned allocations

Right now, dealt with this problem by passing in ``-faligned-allocation`` to compiler flags and added a temporary override for aligned allocation functions to ``zoslib``. Who knows how long they'll stick around, though. 

### Runtime Issues 

In trying to run a basic helloworld example program that grpc provides (see ``examples/cpp/helloworld``), there seemed to be an issue with socket reads and writes giving an ``EWOULDBLOCK`` errno that isn't handled by the logic.


This errno is supposedly (according to the man pages for the relevant functionality) equivalent to ``EAGAIN``, so I made grpc handle ``EWOULDBLOCK`` just like it handles ``EAGAIN``. 

### Other patches

All other fixes should be quite trivial and straightforward. For third party patches, I mainly used those from ``protobufport``. 

### Future Contributions


We should define our own platform string. Right now, we're pretending to be Linux, which seems to be fine, speaking from both build and runtime perspectives, since z/OS has much Linux cross compatibility support beyond the POSIX standard. But, we shouldn't keep doing this. Someone should figure out what configuration macros should be enabled for z/OS.

# Troubleshooting

# Contributing
Contributions are welcome! Please follow the [zopen contribution guidelines](https://github.com/zopencommunity/meta/blob/main/CONTRIBUTING.md).
