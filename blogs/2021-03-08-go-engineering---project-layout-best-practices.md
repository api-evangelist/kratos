---
title: "Go Engineering - Project Layout Best Practices"
url: "https://go-kratos.dev/blog/kratos/go-project-layout/"
date: "Mon, 08 Mar 2021 00:00:00 GMT"
author: ""
feed_url: "https://go-kratos.dev/blog/rss.xml"
---
<div><h2 id="introduction">Introduction</h2></div>
<p>This article mainly discusses some thoughts on <strong>Go project engineering</strong> and the design concepts of <strong>Kratos</strong> from different perspectives of the project.</p>
<p>Go is a language oriented around package names, allowing Go project layouts to be organized through various package names. By following standardized design principles, communication among team members can be greatly improved.</p>
<div><h2 id="project-layout">Project Layout</h2></div>
<p>Every company should establish a unified Kit toolkit project (foundation libraries/framework) and Application project for different microservices.
The foundation library Kit is an independent project, and it is recommended to have only one at the company level. Splitting by functional directories can bring a lot of management overhead, so consolidation is recommended.</p>
<blockquote>
<p>by Package Oriented Design
“To this end, the Kit project is not allowed to have a vendor folder. If any of packages are dependent on 3rd party packages, they must always build against the latest version of those dependences.”</p>
</blockquote>
<div><h3 id="kit-foundation-library">Kit Foundation Library</h3></div>
<p>Treat the Kit project as the company’s standard library, so there should be only one. The Kit foundation library should also have the following characteristics:</p>
<ul>
<li>Simple: Not over-engineered, with straightforward and simple code;</li>
<li>General: Provides the functionality required for general business development;</li>
<li>Efficient: Improves the efficiency of business iterations;</li>
<li>Stable: High testability and coverage of the foundation library, safe and reliable with online practice;</li>
<li>Robust: Reduces misuse through good foundation library design;</li>
<li>High Performance: High performance, but avoids hack optimizations specifically for performance, such as introducing unsafe;</li>
<li>Extensibility: Good interface design for extending implementations, or extending functionality by adding new foundation library directories;</li>
<li>Fault Tolerance: Designed for failure, incorporating a deep understanding of SRE, with high robustness;</li>
<li>Toolchain: Includes a large number of toolchains, such as auxiliary code generation, lint tools, etc.;</li>
</ul>
<p>Taking Kratos as an example, a typical Kit foundation library might look like this:</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>github.com/go-kratos/kratos</span></div></div><div><div><span>├── cmd</span></div></div><div><div><span>├── docs</span></div></div><div><div><span>├── internal</span></div></div><div><div><span>├── examples</span></div></div><div><div><span>├── api</span></div></div><div><div><span>├── errors</span></div></div><div><div><span>├── config</span></div></div><div><div><span>├── encoding</span></div></div><div><div><span>├── log</span></div></div><div><div><span>├── metrics</span></div></div><div><div><span>├── metadata</span></div></div><div><div><span>├── middleware</span></div></div><div><div><span>├── transport</span></div></div><div><div><span>├── registry</span></div></div><div><div><span>├── third_party</span></div></div><div><div><span>├── app.go</span></div></div><div><div><span>├── options.go</span></div></div><div><div><span>├── go.mod</span></div></div><div><div><span>├── go.sum</span></div></div></code></pre></figure></div>
<blockquote>
<p>Note: To ensure the portability of the Kit foundation library, interface abstraction should be performed as much as possible, and the go.mod dependencies on third-party libraries should be kept as simple as possible. Then, extend the foundation library through plugins to meet the customization needs of different businesses.</p>
</blockquote>
<div><h3 id="application-project">Application Project</h3></div>
<p>If you are trying to learn Go, or building a PoC or a toy project for yourself, this project layout is not necessary. Start with something very simple (a main.go file is sufficient). When more people are involved in the project, more structure will be needed, including a Toolkit to facilitate generating project templates and ensuring a unified engineering directory layout as much as possible.</p>
<img alt="kratos ddd" src="https://go-kratos.dev/images/ddd.png" width="500px" />
<p>For example, generating a Go engineering project template using the Kratos tool:</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span># Create project template</span></div></div><div><div><span>kratos new helloworld</span></div></div><div><div>
</div></div><div><div><span>cd helloworld</span></div></div><div><div><span># Pull project dependencies</span></div></div><div><div><span>go mod download</span></div></div><div><div><span># Generate proto template</span></div></div><div><div><span>kratos proto add api/helloworld/helloworld.proto</span></div></div><div><div><span># Generate client source code</span></div></div><div><div><span>kratos proto client api/helloworld/helloworld.proto</span></div></div><div><div><span># Generate server template</span></div></div><div><div><span>kratos proto server api/helloworld/helloworld.proto -t internal/service</span></div></div></code></pre></figure></div>
<p>In Kratos, a typical Go project layout might look like this:</p>
<div><figure><figcaption></figcaption><pre><code><div><div><span>application</span></div></div><div><div><span>|____api</span></div></div><div><div><span>| |____helloworld</span></div></div><div><div><span>| | |____v1</span></div></div><div><div><span>| | |____errors</span></div></div><div><div><span>|____cmd</span></div></div><div><div><span>| |____helloworld</span></div></div><div><div><span>|____configs</span></div></div><div><div><span>|____internal</span></div></div><div><div><span>| |____conf</span></div></div><div><div><span>| |____data</span></div></div><div><div><span>| |____biz</span></div></div><div><div><span>| |____service</span></div></div><div><div><span>| |____server</span></div></div><div><div><span>|____test</span></div></div><div><div><span>|____pkg</span></div></div><div><div><span>|____go.mod</span></div></div><div><div><span>|____go.sum</span></div></div><div><div><span>|____LICENSE</span></div></div><div><div><span>|____README.md</span></div></div></code></pre></figure></div>
<div><h3 id="application-types">Application Types</h3></div>
<p>The app service types in microservices are mainly divided into five categories: interface, service, job, admin, and task. The application cmd directory is responsible for the program’s: startup, shutdown, configuration initialization, etc.</p>
<ul>
<li>interface: External BFF service that accepts requests from users, such as exposing HTTP/gRPC interfaces.</li>
<li>service: Internal microservice that only accepts requests from other internal services or gateways, such as exposing gRPC interfaces only for internal services.</li>
<li>admin: Different from service, more oriented towards operational services, usually with higher data permissions, isolation brings better code-level security.</li>
<li>job: Streaming task processing service, with upstream generally depending on message broker.</li>
<li>task: Scheduled tasks, similar to cronjobs, deployed to task hosting platforms.</li>
</ul>
<div><h2 id="application-directories">Application Directories</h2></div>
<div><h3 id="cmd">/cmd</h3></div>
<p>The main application for this project.
The directory name for each application should match the name of the executable you want (e.g., <code dir="auto">/cmd/myapp</code>).
Do not place too much code in this directory. If you think the code can be imported and used in other projects, then it should be located in the <code dir="auto">/pkg</code> directory. If the code is not reusable, or you do not want others to reuse it, place that code in the <code dir="auto">/internal</code> directory.</p>
<div><h3 id="internal">/internal</h3></div>
<p>Private application and library code. This is code that you do not want others to import into their applications or libraries. Note that this layout pattern is enforced by the Go compiler itself. For more details, see the Go 1.4 release notes. Note that you are not limited to the top-level <code dir="auto">internal</code> directory. You can have multiple internal directories at any level of the project tree.
You can choose to add some additional structure to the <code dir="auto">internal</code> package to separate shared and non-shared internal code. This is not required (especially for smaller projects), but it is good to have visual clues showing the intended use of the package. Your actual application code can be placed in the <code dir="auto">/internal/app</code> directory (e.g., <code dir="auto">/internal/app/myapp</code>), and the code shared by these applications can be placed in the <code dir="auto">/internal/pkg</code> directory (e.g., /internal/pkg/myprivlib).
Because we are accustomed to integrating related services, such as the account service, which internally includes rpc, job, admin, etc., after integrating related services, it is necessary to distinguish apps. For a single service, <code dir="auto">/internal/myapp</code> can be omitted.</p>
<div><h3 id="pkg">/pkg</h3></div>
<p>Library code that can be used by external applications (e.g., <code dir="auto">/pkg/mypubliclib</code>). Other projects will import these libraries, so think twice before putting things here :-) Note that the <code dir="auto">internal</code> directory is a better way to ensure private packages are not importable, as it is enforced by Go. The <code dir="auto">/pkg</code> directory is still a good way to explicitly indicate that the code in this directory is safe for others to use.</p>
<blockquote>
<p>Inside the /pkg directory, you can refer to the organization of the Go standard library, categorized by functionality. /internal/pkg is generally used for public shared code across multiple applications within a project, but its scope is limited to a single project.</p>
</blockquote>
<p>The blog post I’ll take pkg over internal by Travis Jeffery provides a good overview of the <code dir="auto">pkg</code> and <code dir="auto">internal</code> directories and when it makes sense to use them.
This is also a way to group Go code in one place when the root directory contains a large number of non-Go components and directories, making it easier to run various Go tools in an organized manner.</p>
<div><h2 id="service-application-directories">Service Application Directories</h2></div>
<div><h3 id="api">/api</h3></div>
<p>API protocol definition directory, services.proto protobuf files, and generated go files. We usually describe the API documentation directly in the proto file.</p>
<div><h3 id="configs">/configs</h3></div>
<p>Configuration file templates or default configurations.</p>
<div><h3 id="test">/test</h3></div>
<p>Additional external test applications and test data. You can structure the /test directory as needed. For larger projects, it makes sense to have a data subdirectory. For example, you can use /test/data or /test/testdata (if you need to ignore the contents of the directory). Note that Go also ignores directories or files starting with ”.” or ”_”, so there is more flexibility in how you name test data directories.</p>
<div><h2 id="service-internal-directories">Service Internal Directories</h2></div>
<p>Under the Application directory, there are api, cmd, configs, internal, pkg directories. README, CHANGELOG, OWNERS are usually placed inside.
internal is to avoid cross-directory references to internal structs like data, biz, service, server within the same business.</p>
<div><h3 id="data">data</h3></div>
<p>Business data access, including cache, db, etc. encapsulation, implementing the repo interface of biz. We might confuse data with dao; data emphasizes business meaning. What it needs to do is to retrieve the domain object again. We have removed the DDD infra layer.</p>
<div><h3 id="biz">biz</h3></div>
<p>Business logic assembly layer, similar to the domain layer in DDD. data is similar to the repo in DDD, where the repo interface is defined, using the dependency inversion principle.</p>
<div><h3 id="service">service</h3></div>
<p>Implements the service layer defined by the api, similar to the application layer in DDD, handling the conversion from DTO to biz domain entities (DTO -> DO), and coordinating interactions among various biz, but should not handle complex logic.</p>
<div><h3 id="server">server</h3></div>
<p>Creation and configuration of http and grpc instances, and registration of the corresponding service.</p>
<div><h2 id="not-recommended-directories">Not Recommended Directories</h2></div>
<div><h3 id="src"><del>src/</del></h3></div>
<p>The src directory is a common pattern in Java development projects, but in Go development projects, try to avoid using the src directory.</p>
<div><h3 id="model"><del>model/</del></h3></div>
<p>In other languages, a very common module is called model, where all types are placed in model. However, this is not recommended in Go because Go’s package design is based on functional responsibilities. For example, a User model should be declared in the functional module where it is used.</p>
<div><h3 id="xxs"><del>xxs/</del></h3></div>
<p>Directories or packages with plural names. Although there is a strings package in the Go source code, singular forms are more commonly used.</p>
<div><h2 id="summary">Summary</h2></div>
<p>In actual Go project development, it is essential to apply flexibility. Of course, you can also completely disregard such architectural layering and package design rules. Everything depends on the project size, business complexity, the breadth and depth of personal professional skills, and time constraints.</p>
<p>Moreover, it is crucial to choose a Kit foundation library suitable for your team based on the actual situation, conduct thorough research, and determine whether it can meet plugin customization needs. Maintain the team’s Kit foundation library and code standards, and encourage developers to actively participate and contribute.</p>
<p>If anyone has better architectural design concepts, welcome to discuss them in the go-kratos community. I hope this article is helpful to you~</p>
<div><h2 id="references">References</h2></div>
<ul>
<li><a href="https://www.ardanlabs.com/blog/2017/02/package-oriented-design.html">Package Oriented Design</a></li>
<li><a href="https://docs.microsoft.com/en-us/previous-versions/msp-n-p/ee658109(v=pandp.10)">Layered Application Guidelines</a></li>
<li><a href="https://github.com/golang-standards/project-layout">Standard Go Project Layout</a></li>
<li><a href="https://github.com/danceyoung/paper-code/blob/master/package-oriented-design/packageorienteddesign.md">Go Package-Oriented Design and Architectural Layering</a></li>
<li><a href="https://u.geekbang.org/subject/go">Go Advanced Training Camp - Geek Time</a></li>
</ul>
