file-part-upload
================

## 安装和配置

### 引入 gem

```ruby
# Gemfile
gem 'file-part-upload', 
  :git => 'git://github.com/mindpin/file-part-upload.git',
  :tag => '1.0.6'
```

## 使用说明

### 配置文件存储路径和 url

绝对路径
```ruby
# initialize/file_part_upload.rb
FilePartUpload.config do
  # 最后的硬盘路径 /files/:id/file/:name
  # url 的 path 部分是 /files/:id/file/:name
  path => '/files/:id/file/:name'
end
```

相对路径
```ruby
# initialize/file_part_upload.rb
FilePartUpload.config do
  # 最后的硬盘路径 :project_root/public/files/:id/file/:name
  # url 的 path 部分是 /files/:id/file/:name
  path => 'files/:id/file/:name'
end
```

自定义url
```ruby
# initialize/file_part_upload.rb
FilePartUpload.config do
  # 最后的硬盘路径 :project_root/public/files/:id/file/:name
  # url 的 path 部分是 /attachment/:id/file/:name
  path => 'files/:id/file/:name'
  url  => '/attachment/:id/file/:name'
end
```

### 配置缩略图
```ruby
# initialize/file_part_upload.rb
FilePartUpload.config do
  image_version :large do
    process :resize_to_fill => [180, 180]
  end
  image_version :normal do
    process :resize_to_fill => [64, 64]
  end
  image_version :small do
    process :resize_to_fill => [30, 30]
  end

  image_version :xxx do
    process :resize_to_fit => [30, 31]
  end
end
```

重新生成缩略图
```ruby
file_entity.attach.resize!
```

### 分段上传一个文件

```ruby
  # 第一步
  file_entity = FileEntity.new(:attach_file_name => file_name, :attach_file_size => @file_size)
  file_entity.save
  
  # 后续步骤
  # blob 是 file 对象
  blobs.each do |blob|
    file_entity.save_blob(blob)
  end
  
  
  # 是否上传完毕
  file_entity.uploaded?
  
  # 是否还没有上传完毕
  file_entity.uploading?
  
  # 原始文件路径
  file_entity.attach.path

  # 缩略图文件路径
  file_entity.attach.path(:small)
  
  # 原始文件url
  file_entity.attach.url

  # 缩略图文件url
  file_entity.attach.url(:small)
  
  # 文件大小
  file_entity.attach.size
  
  # 文件 content_type
  file_entity.attach.content_type
  
  # 获取文件原始名
  file_entity.attach_file_name
  
  # 已经保存的文件大小
  file_entity.saved_size

```

### 整个文件上传

```ruby
  file_entity = FileEntity.new(:attach => file)
  file_entity.save
```


### 配置给 file_entity include module

```ruby
# initialize/file_part_upload.rb
module ExpandMethods
  # 给 file_entity 增加一些方法
end
FilePartUpload.config do
  add_methods ExpandMethods
end
```

### rails controller/sinatra action 引入 module
```ruby
class ApplicationController < ActionController::Base
  include FilePartUpload::ControllerHelper
end

class XXXApp < Sinatra::Base
  helpers FilePartUpload::ControllerHelper
end
```

FilePartUpload::ControllerHelper 的实现
```ruby
module FilePartUpload
  module ControllerHelper
    def start_uploading(file_name, file_size)
      file_entity = FilePartUpload::FileEntity.new(:attach_file_name => file_name, :attach_file_size => file_size)
      file_entity.save
    end

    def continue_uploading(id, blob)
      file_entity = FilePartUpload::FileEntity.find(id)
      file_entity.save_blob(blob)
    end

    def full_upload(file)
      file_entity = FilePartUpload::FileEntity.new(:attach => file)
      file_entity.save
    end
  end
end
```

### office 相关

```
# 判断是否是 office
file_entity.is_office?
```

controller 或者 view include FilePartUpload::OfficeHelper
```
# 传入一个 office 文件的在线url,生成一个可以在线预览该文件的 url
office_online_view_url(office_file_url)

# 传入一个 office 文件的 file_entity,生成一个可以在线预览该文件的 url
office_online_view_url(file_entity)
```