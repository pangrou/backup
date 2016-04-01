---
title: ios基础学习
date: 2016-03-16 17:52:17
tags:
---

swift实现一个简单的四则运算计算器
```cpp
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var resultLable: UILabel!
    
    var operand1: String = ""
    var operand2: String = ""
    var operator3: String = ""
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    @IBAction func didClicked(sender: AnyObject) {
        let value = sender.currentTitle
        
        if value == "+" || value == "-" || value == "x" || value == "/" {
            operator3 = value!!
            return
        }else if value == "=" {
            var result = 0
            switch operator3 {
                case "+":
                    result = Int(operand1)! + Int(operand2)!
                case "-":
                    result = Int(operand1)! - Int(operand2)!
                case "x":
                    result = Int(operand1)! * Int(operand2)!
                case "/":
                    result = Int(operand1)! / Int(operand2)!
                default:
                    result = 0
            }
            
            resultLable.text = "\(result)"
            
            return
        }
        
        if operator3 == "" {
            operand1 += value!!
        }else {
            operand2 += value!!
        }
    }

}

```

用户登录
```cpp
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var usernameLable: UITextField!
    @IBOutlet weak var passwdLbale: UITextField!
    @IBOutlet weak var tipLable: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
//        print(usernameLable.text)
//        print(passwdLbale.text)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    @IBAction func didclicked(sender: UIButton) {
        let username = usernameLable.text
        let passwd = passwdLbale.text
        
        //admin 6666666
        if username == "admin" && passwd == "666666" {
            tipLable.text = "login success!"
        }else {
            tipLable.text = "username or passwd error!"
        }
    
    }

}
```
根据成绩更换界面照片
```cpp
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var sorceTextFiled: UITextField!
    @IBOutlet weak var imageview: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    @IBAction func clicked(sender: AnyObject) {
        var score = Int(sorceTextFiled.text!)
        switch score! {
            case 0...59:
                imageview.image = UIImage(named: "s08")
                break
            case 60...79:
                imageview.image = UIImage(named: "s09")
                break
            case 80...100:
                imageview.image = UIImage(named: "s10")
                break
            default:
                print("wrong score!")
                break
        }
    }

}
```
简单记事本
```cpp

import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var noteTextView: UITextView!
    @IBOutlet weak var viewTextFiled: UILabel!
    @IBOutlet weak var nameTextFiled: UITextField!

//    var array = [String]()
    var array = [Dictionary<String,String>]()

    override func viewDidLoad() {
        
        super.viewDidLoad()
        // Do any additional setup
//        after loading the view, typically from a nib.
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    @IBAction func didclicked(sender: UIButton) {
        var content = noteTextView.text
//        array.append(content)
        var name = nameTextFiled.text
        //key-value
        var dict = Dictionary<String,String>()
        dict["name"] = name
        dict["content"] = content
        array.append(dict)
//        var dict2: Dictionary<String,Int> = ["age": 18]
        
    }
    
    @IBAction func viewClicked(sender: UIButton) {
//        self.view.endEditing(false)
        for var value = 0; value < array.count; value++ {
            var dict = array[value]
            var name = dict["name"]
            viewTextFiled.text! += name! + " "
            print(dict["content"])
        }
        
    }
    
    @IBAction func openClicked(sender: UIButton) {
        var name = self.nameTextFiled.text
        for dict in array {
            var filename = dict["name"]
            if name == filename {
                noteTextView.text = dict["content"]!
                break
            }
        }
    }

}
```