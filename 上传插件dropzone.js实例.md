dropzone.js默认是Ajax上传图片给服务器，那么如何获取到图片名呢？其实我们是可以通过dropzone的success函数获取到服务器返回的数据

dropzone.js在HTML的配置如下：

	Dropzone.autoDiscover = false;//防止报"Dropzone already attached."的错误
        $(".dropzone").dropzone({
        url: "__URL__/upload/",
        addRemoveLinks: true,
        dictRemoveLinks: "x",
        dictCancelUpload: "x",
        paramName:"userImg",
        maxFiles: 10,
        maxFilesize: 5,
        acceptedFiles: "image/*",
        init: function() {

            //res为服务器响应回来的数据
            this.on("success", function(file, res) {

                //将json字符串转换成json对象
                var obj = JSON.parse(res);

                console.log(obj);
                
	                if( obj.status == 200 ){

                    //将服务器得到的数据生成一个隐藏域。做商品添加的时候就可以获取到了
                    var input = '<input type="hidden" name="'+obj.details.savename+'" value="'+obj.details.savepath+obj.details.savename+'" />';
                    $('.myform').append(input);

                }else{
                    alert('上传失败');
                }
                

                
            });
            this.on("removedfile", function(file) {
                console.log("File " + file.name + "removed");
            });
        }
    });

PHP的代码如下(Thinkphp代码)：

	public function upload()
    {    

        /*
          添加商品 ：商品名、商品图片
         */

           // 实例化上传类    
          $upload = new \Think\Upload();

           // 设置附件上传大小    
          $upload->maxSize   =     3145728 ;

        // 设置附件上传类型   
          $upload->exts      =     array('jpg', 'gif', 'png', 'jpeg');

	//A开发者写了upload()   B开发
          // 设置附件上传目录   
           $upload->savePath  =      './Public/Uploads/'; 


          //返回上传信息
          $info   =   $upload->uploadOne($_FILES['userImg']);   
          // dump($info);exit;
          if( !$info ) {
            // 上传错误提示错误信息
              // $this->error($upload->getError()); 
              $data['status'] = 404;

              //错误信息
              $data['msg']    = $upload->getError();
              
              echo  json_encode($data);

          }else{
            // 上传成功 (图片路径、图片名字)
              
              $data['status']  = 200;
              $data['msg']     = 'UPLOAD SUCCESS';

              //图片原始名字
              $data['details']['originName'] = $info['name'];
              $data['details']['savename'] = $info['savename'];
              $data['details']['savepath'] = $info['savepath'];

              echo json_encode($data);
          }
    }


