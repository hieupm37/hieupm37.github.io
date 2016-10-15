---
layout: post
title: "Scoped object"
categories: cpp
---
When programming with C++, you usually meet a pair of acquire/release resources (memory, file, mutex, etc) or a pair of change/restore state (or setting). For examples:

{% highlight cpp %}
void foo() {
  FILE* file = fopen("path_to_file", "w");  // Acquire a file resource

  // Do something with |file|

  fclose(file);                             // Release resource
}

void draw_with_opengl() {
  glPushMatrix();  // Save and change matrix state

  glRotated(...);  // Change matrix
  // Drawing

  glPopMatrix();   // Restore matrix state to previous
}
{% endhighlight %}

The resource will be leaked, the state won't restore if:

* Someone (maybe you) accidentally exits the function between the pair.
* There is an exception between the pair. Then the code is not exception safe.

Because the destructor will be called when an object is destroyed, we can use it to release the resource automatically. This object is called scoped object.

{% highlight cpp %}
class ScopedFile {
 public:
  explicit ScopedFile(FILE* file) : file_(file) {}
  ~ScopedFile() { fclose(file_); }

  FILE* raw() { return file_; }

 private:
  FILE* file_;
};

void foo() {
  ScopedFile scoped_file(fopen("path_to_file", "w"));

  // Do something with file via scoped_file->raw()
}  // file will automatically close when |scoped_file| is destroyed.

class ScopedGLMatrix {
  public:
  ScopedGLMatrix() { glPushMatrix(); }
  ~ScopedGLMatrix() { glPopMatrix(); }
};

void draw_with_opengl() {
  ScopedGLMatrix save_gl_matrix;

  glRotated(....);  // Change matrix
  // Drawing
}  // GL matrix will automatically restore when |save_gl_matrix| is destroyed
{% endhighlight %}

By using scoped object, you can prevent resource leak, the code is less error-prone and exception safe. Using it widely in your project is a good habit.

P/S: Smart pointer is a kind of scoped object. std::unique_ptr, std::shared_ptr and std::weak_ptr are smart pointers in C++11.
