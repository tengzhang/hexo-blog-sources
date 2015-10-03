title: gitlab只使用https协议
comment: true
toc: true
share: true
date: 2015-05-30 22:25:04
tags: [gitlab]
---
主要修改两个地方

<!-- more -->

### 1 修改`app/views/shared/_clone_panel.html.haml`
删掉下面这段代码
``` ruby
%button{ |
      class: "btn #{ 'active' if default_clone_protocol == 'ssh' }#{ ' has_tooltip' if current_user && current_user.require_ssh_key? }", |
      :"data-clone" => project.ssh_url_to_repo, |
      :"data-title" => "Add an SSH key to your profile<br> to pull or push via SSH",
      :"data-html" => "true",
      :"data-container" => "body"}
      SSH
```

### 2 修改`./app/helpers/projects_helper.rb`
``` ruby
def default_url_to_repo(project = nil)
    project = project || @project
    current_user ? project.http_url_to_repo : project.url_to_repo
  end

  def default_clone_protocol
    current_user ? "http" : "ssh"
  end
```