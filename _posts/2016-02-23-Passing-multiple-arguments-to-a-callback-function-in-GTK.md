---

layout: post
title: GTK+中回调传递多个参数
category : C
tags : [C, GTK, GUI]

---

在GTK+中，绑定信号的方式一般如下：

`g_signal_connect (G_OBJECT (button), "clicked", G_CALLBACK(button_clicked), # an argument #);`

大多数情况下，传递一个参数就够用了。但还是有些情况会遇到比较大的麻烦。

最近在写Cplayer-gui的时候，在批量添加文件这一块儿就遇到问题。

在文件选择dialog弹出来后，用户鼠标批量选择文件（或直接ctrl+a），然后点击"add"按钮。这时候，触发"button-clicked"信号。

刚开始的时候，我这样传递的参数：

    g_signal_connect(button, "clicked", G_CALLBACK(file_chooser_ok_button), file_chooser_dialog);

把文件选择器所在的dialog传递到回调函数中。回调函数做大概如下动作：

    void file_chooser_ok_button( GtkButton *button, gpointer file_chooser)
    {
        GSList *list;
        list = gtk_file_chooser_get_filenames (GTK_FILE_CHOOSER (file_chooser));
        g_slist_foreach (list, add_file, NULL);
        g_slist_free(list);
        gtk_widget_hide (GTK_WIDGET(file_chooser));
    }

即：从文件选择dialog中，拿到全部被选中的文件路径列表，循环这个列表给gtk liststore中添加数据。

可是最上层的信号链接函数只能传递一个参数到回调函数中，从这里拿不到liststore，所以"add_file"函数自然得不到这个参数。

google搜之，找到比较不错的解决办法：外层信号绑定传一个自定义的GObject进去。

用GLib的g_object_set_data和g_object_get_data可以给一个对象中添加属性，也算是C语言下的OOP的实现方式= =！

首先，创建一个GObject *， 然后填充数据。在这里因为使用g_object_new()来构建自己的Object不是很必要，所以直接偷懒创建了
一个label取代之。。

**注意！空的GObject不能使用到这里！会被G_IS_OBJECT拦截的！

    GtkWidget *user_data = gtk_label_new("this is a new lable");
    g_object_set_data(G_OBJECT(user_data), "file_chooser", file_chooser_dialog);
    g_object_set_data(G_OBJECT(user_data), "liststore", liststore);
    g_signal_connect(button, "clicked", G_CALLBACK(file_chooser_ok_button), user_data);

回调中这样处理就可以轻松将liststore传递给下一层回调了：

    void file_chooser_ok_button(GtkButton *button, GObject *user_data) {
        GSList *list;
        GObject *file_chooser = g_object_get_data(user_data, "file_chooser");
        GObject *liststore = g_object_get_data(user_data, "liststore");
        list = gtk_file_chooser_get_filenames(GTK_FILE_CHOOSER(file_chooser));
        g_slist_foreach(list, add_song, liststore);
        g_slist_free(list);
        gtk_widget_hide(GTK_WIDGET(file_chooser));
    }
