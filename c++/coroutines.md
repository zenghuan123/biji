* 协程方法的局部变量是按值拷贝,指针和引用需要注意是否已经失效了
* 例子
```

#include <coroutine>
#include <iostream>
struct promise;
struct coroutine {
    using promise_type = ::promise;
    using handle_type = std::coroutine_handle<promise_type>;
    handle_type handler_;
    void resume() {
        //继续执行
        handler_.resume();
    }
    void destroy() {
        handler_.destroy();
    }
};

struct promise
{
    promise() {
        std::cout << " promis()" << std::endl;
    }
    coroutine get_return_object() { 
        std::cout <<__FUNCTION__ << std::endl;
        //调用一个协程的方法时 调用这个方法
        return { std::coroutine_handle<promise>::from_promise(*this) };
    }
    std::suspend_never initial_suspend() noexcept {
        //get_return_object调用会玩会立即调用这个方法
        //返回std::suspend_always表示创建玩coroutine后就挂起
        //如果返回std::suspend_never创建完之后会立即执行方法
        std::cout << __FUNCTION__ << std::endl;
        return {}; 
    }
    std::suspend_always final_suspend() noexcept {
        //执行co_return时调用，协程方法执行到最后也会调用
        //出现异常时也会调用
        //handler_.destroy();时不会调用
        std::cout << __FUNCTION__ << std::endl;
        return {};
    }
    std::suspend_always yield_value(int i) {
        //co_yield 变量时调用，参数可以是任意参数
        //可以将i保存到 promise对象上，这样调用方就能获取到值
        std::cout << __FUNCTION__ << std::endl;
        return {};
    }
    void return_value(int i){
        //这个是co_return i;时调用到
    }
    void return_void() {
        //这个是co_return; 无参数时调用到
        std::cout << __FUNCTION__ << std::endl;
    }
    void unhandled_exception() {
        //协程方法以出现异常的状态下退出时调用到
        //可以这样保存当前异常exception_ = std::current_exception(); 
        //之后也会调用final_suspend
        std::cout << __FUNCTION__ << std::endl;
        
    }
    struct awaitable {
        bool _suspend;
        bool await_ready() const noexcept {
            // 返回false表示执行要挂起
            // 返回true表示不挂起继续执行
            return _suspend;
        };
        void await_suspend(std::coroutine_handle<> h) {

        }
        void await_resume() {

        }
    };
    awaitable await_transform(int i) {
        std::cout << " i is " << i << std::endl;
        return { false };
    }
    awaitable await_transform(std::string_view msg) {
        std::cout << " msg is " << msg << std::endl;
        return {true};
    }
    template<typename awaitable_type,typename funtion_type= decltype(awaitable_type::await_ready)>
    awaitable_type await_transform(awaitable_type obj) {
        return obj;
    }
};
auto switch_to_new_thread(std::jthread& out)
{
    struct awaitable
    {
        std::jthread* p_out;
        bool await_ready() {
            std::cout << __FUNCTION__ << std::endl;
            return false;
        }
        void await_suspend(std::coroutine_handle<> h)
        {
            std::cout << __FUNCTION__<<" start " <<std::this_thread::get_id()<<  std::endl;
            std::jthread& out = *p_out;
            if (out.joinable())
                throw std::runtime_error("Output jthread parameter not empty");
            out = std::jthread([h] {
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << " thread resume " << std::this_thread::get_id() << std:: endl;
                h.resume();//执行co_yield 1
                h.resume();//执行co_return
                std::cout << " thread end " << std::this_thread::get_id() << std::endl;
                });
            std::cout << __FUNCTION__ << " end " << std::endl;
        }
        void await_resume() {
            std::cout << __FUNCTION__ <<" "<< std::this_thread::get_id() << std::endl;
        }
    };
    return awaitable{ &out };
}

coroutine run_coroutinue(int i, std::jthread& out) {
    std::cout << __FUNCTION__ << " start " << std::endl;
    co_await 1;
    co_await "abcdef";
    co_await switch_to_new_thread(out);//线程内部调用resume了
    co_yield 1;
    std::cout << __FUNCTION__ << " end "<<std::this_thread::get_id() << std::endl;//这里是新创建的线程运行到
    co_return;//调用这个后会调用final_suspend
    std::cout << __FUNCTION__ << " not run " << std::endl;
}

int main(int argc,char* argv[]) {
    std::jthread out;
    std::cout << "start create coroutine " <<std::this_thread::get_id()<< std::endl;
    coroutine h = run_coroutinue(12, out); //创建协程对象，局部参数都是按值拷贝的。
    //h.destroy();//调用destroy是不会触发
    std::cout << "after create coroutine " << std::endl;
    h.resume();
    std::cout << "after resume1 coroutine " << std::endl;
    out.join();
    std::cout << "thread leave " << std::endl;
    h.destroy();
    std::cout << "after resume2 destroy " << std::endl;
    return 0;
}
```
*co_yield相当于 co_await promise.yield_value(expr)