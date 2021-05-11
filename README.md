# devops-netology
## .gitignore

### Исключаются директории .terraform в любой вложенной директории кроме корневой / (причем со всеми вложенными файлами и поддиректориями)
```**/.terraform/*```

### Исключаются  файлы с расширением .tfstate и любым названием до точки, а также файлы с любым названием до первой точки и после второй точки в корневой и любых вложенных директориях. Т.е. будут исключены, например, файлы .tfstate, abc.tfstate, .tfstate., abc.tfstate., abc.tfstate.def и т.д.
```
*.tfstate
*.tfstate.*
```

### Исключаются файлы с названием crash.log в корневой и любых вложенных директориях
```crash.log```

### Исключаются  файлы с расширением .tfvars и любым названием до точки в корневой и любых вложенных директориях (.tfvars, qwe.tfvars и т.д.)
```*.tfvars```

### Исключаются файлы с названием override.tf и override.tf.json, а также файлы _override.tf, asd_override.tf, _override.tf.json, dfg_override.tf.json и т.д. в корневой и любых вложенных директориях
```
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```
### Исключение ниже игнорируется, т.к. впереди стоит \#
```# !example_override.tf```

### Исключение ниже игнорируется, т.к. впереди стоит \#
```# example: *tfplan*```

### Исключаются файлы с названием .terraformrc и terraform.rc в корневой и любых вложенных директориях
```
.terraformrc
terraform.rc
```
### New line
