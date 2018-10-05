# detectron-caffe2-tutorial
learning to write custom opertator for detectron/caffe2

/*  A struct that abstracts on top of dense and sparse blobs.
 *
 * For a dense blob, its gradient name should be written into dense_, and for
 * a sparse blob, its gradient name should be written into indice_ for
 * the sparse indices and value_ for the values.
 */ 
 
 /src/main/cpp/caffe2/core/operator_gradient.h
 class GradientMakerBase{...}
  
  // Helper functions to return names for the gradient computation.
  // I(idx), O(idx): return the input and output names.
  // GO(idx): return the name of the gradient for output idx.
  // GI(idx), GI_I(idx), GI_V(idx): return the name of the gradient for
  //     input idx, and also registers that name into the gradient
  //     registry to be returned.
  string I(const int i) {
    CAFFE_ENFORCE((i >= 0) && (i < def_.input().size()));
    return def_.input(i);
  }
  string O(const int i) {
    CAFFE_ENFORCE((i >= 0) && (i < def_.output().size()));
    return def_.output(i);
  }
  string GI(const int i) {
    CAFFE_ENFORCE(
        !g_input_.at(i).IsSparse(),
        "Input ",
        def_.input(i),
        " already set to sparse.");
    g_input_.at(i).dense_ = GradientName(def_.input(i));
    return GradientName(def_.input(i));
  }
  string GI_I(const int i) {
    CAFFE_ENFORCE(
        !g_input_.at(i).IsDense(),
        "Input ",
        def_.input(i),
        " already set to dense.");
    g_input_.at(i).indices_ = GradientSliceIndices(def_.input(i));
    return GradientSliceIndices(def_.input(i));
  }
  string GI_V(const int i) {
    CAFFE_ENFORCE(
        !g_input_.at(i).IsDense(),
        "Input ",
        def_.input(i),
        " already set to dense.");
    g_input_.at(i).values_ = GradientSliceValues(def_.input(i));
    return GradientSliceValues(def_.input(i));
  }
  string GO(const int i) {
    CAFFE_ENFORCE(
        g_output_.at(i).IsDense(),
        "Gradient of output ",
        def_.output(i),
        (g_output_.at(i).IsSparse() ? " is sparse (expected dense)."
                                    : " is not provided!"));
    return g_output_.at(i).dense_;
  }
  string GO_I(const int i) {
    CAFFE_ENFORCE(
        g_output_.at(i).IsSparse(),
        "Gradient of output ",
        def_.output(i),
        (g_output_.at(i).IsDense() ? " is dense (expected sparse)."
                                   : " is not provided!"));
    return g_output_.at(i).indices_;
  }
  string GO_V(const int i) {
    CAFFE_ENFORCE(
        g_output_.at(i).IsSparse(),
        "Gradient of output ",
        def_.output(i),
        (g_output_.at(i).IsDense() ? " is dense (expected sparse)."
                                   : " is not provided!"));
    return g_output_.at(i).values_;
  }
  const GradientWrapper& GradOut(int i) {
    return g_output_.at(i);
  }
  
  
    /**
   * @brief a helper function to allow one to create one single operator
   * def, which is usually the case for many simple operators.
   */
  template <class... Args>
  inline static vector<OperatorDef> SingleGradientDef(const Args&... args) {
    return **vector<OperatorDef>{CreateOperatorDef(args...)};
  }
  
