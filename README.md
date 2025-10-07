#include <bits/stdc++.h>
#include <cmath>
using namespace std;

map<string,double> variables;

bool isOperator(char c) {
    return c=='+' || c=='-' || c=='*' || c=='/' || c=='^';
}

int precedence(char op) {
    if(op=='+' || op=='-') return 1;
    if(op=='*' || op=='/') return 2;
    if(op=='^') return 3;
    return 0;
}

double applyOp(double a, double b, char op){
    switch(op){
        case '+': return a+b;
        case '-': return a-b;
        case '*': return a*b;
        case '/': return a/b;
        case '^': return pow(a,b);
    }
    return 0;
}

double evaluateFunction(const string &func, double val){
    if(func=="sin") return sin(val);
    if(func=="cos") return cos(val);
    if(func=="tan") return tan(val);
    if(func=="asin") return asin(val);
    if(func=="acos") return acos(val);
    if(func=="atan") return atan(val);
    if(func=="sinh") return sinh(val);
    if(func=="cosh") return cosh(val);
    if(func=="tanh") return tanh(val);
    if(func=="sqrt") return sqrt(val);
    if(func=="log") return log(val);
    if(func=="exp") return exp(val);
    throw runtime_error("Función desconocida: " + func);
}

// Evaluar expresión básica sin paréntesis
double evaluateBasic(string expr);

// Evaluar expresión completa
double evaluate(string expr){
    // Eliminar espacios
    expr.erase(remove(expr.begin(), expr.end(), ' '), expr.end());
    // Manejo de asignación de variables
    size_t eq = expr.find('=');
    if(eq!=string::npos){
        string var = expr.substr(0,eq);
        string valueExpr = expr.substr(eq+1);
        double val = evaluateBasic(valueExpr);
        variables[var] = val;
        return val;
    } else return evaluateBasic(expr);
}

double evaluateBasic(const string &expr){
    stack<double> values;
    stack<char> ops;
    size_t i=0;
    while(i<expr.size()){
        if(expr[i]=='('){ ops.push('('); i++; continue; }
        else if(isdigit(expr[i]) || expr[i]=='.'){
            size_t j=i;
            while(j<expr.size() && (isdigit(expr[j])||expr[j]=='.')) j++;
            values.push(stod(expr.substr(i,j-i)));
            i=j;
        }
        else if(isalpha(expr[i]) || expr[i]=='_'){
            size_t j=i;
            while(j<expr.size() && (isalpha(expr[j]) || isdigit(expr[j]) || expr[j]=='_')) j++;
            string token = expr.substr(i,j-i);
            i=j;
            if(i<expr.size() && expr[i]=='('){ // Función
                size_t k=i+1, count=1;
                while(k<expr.size() && count>0){
                    if(expr[k]=='(') count++;
                    else if(expr[k]==')') count--;
                    k++;
                }
                string inside = expr.substr(i+1, k-i-2);
                values.push(evaluateFunction(token, evaluate(inside)));
                i=k;
            } else { // Variable o constante
                if(token=="pi") values.push(M_PI);
                else if(token=="e") values.push(M_E);
                else if(token=="_") values.push(variables["_"]);
                else if(variables.count(token)) values.push(variables[token]);
                else throw runtime_error("Variable desconocida: "+token);
            }
        }
        else if(isOperator(expr[i])){
            while(!ops.empty() && precedence(ops.top())>=precedence(expr[i])){
                double b=values.top(); values.pop();
                double a=values.top(); values.pop();
                char op=ops.top(); ops.pop();
                values.push(applyOp(a,b,op));
            }
            ops.push(expr[i]);
            i++;
        }
        else if(expr[i]==')'){
            while(!ops.empty() && ops.top()!='('){
                double b=values.top(); values.pop();
                double a=values.top(); values.pop();
                char op=ops.top(); ops.pop();
                values.push(applyOp(a,b,op));
            }
            if(!ops.empty()) ops.pop();
            i++;
        }
        else throw runtime_error("Caracter desconocido: "+string(1,expr[i]));
    }
    while(!ops.empty()){
        double b=values.top(); values.pop();
        double a=values.top(); values.pop();
        char op=ops.top(); ops.pop();
        values.push(applyOp(a,b,op));
    }
    return values.top();
}

int main(){
    cout << "=== Calculadora Científica Avanzada con Variables ===\n";
    cout << "Funciones: sin, cos, tan, asin, acos, atan, sinh, cosh, tanh, sqrt, log, exp\n";
    cout << "Constantes: pi, e\n";
    cout << "Variables: x=..., y=..., _ para último resultado\n";
    cout << "Escribe 'q' para salir\n";

    string input;
    vector<string> history;
    while(true){
        cout << ">> ";
        getline(cin,input);
        if(input=="q") break;
        try{
            double result = evaluate(input);
            variables["_"] = result;
            cout << fixed << setprecision(6) << result << "\n";
            history.push_back(input + " = " + to_string(result));
        } catch(const exception &e){
            cout << "Error: " << e.what() << "\n";
        }
    }

    cout << "\n=== Historial de la sesión ===\n";
    for(auto &line: history) cout << line << "\n";

    return 0;
}
