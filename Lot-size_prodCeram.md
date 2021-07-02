# Otimizacao_linear
Projeto desenvolvido na disciplina de Otimização Linear no mestrado do Programa de Pós-Graduação de Engenharia de Produção e Sistema da UFPB

using JuMP
using Cbc
using PrettyTables

model = Model(Cbc.Optimizer)

T = 3; #periodos
P = 3; #produtos

@variable(model, x[i=1:P,j=1:T] >= 0) #quantidade a ser produzido de produtos m2
@variable(model, e[i=1:P,j=1:T] >= 0) #estoque de produtos m2


#criar a matriz das instancias
demandaf = [36960 28980 21470; 11490 28670 13130; 11560 12330 13460]
disponibilidade = [42600; 42240; 42600]

cf = [16.00	16.50 18.43; 18.08 18.48 20.82; 18.98 19.22 22.24]
hf = [0.64	0.66 0.37; 0.64	0.66 0.37; 0.64	0.66 0.37]
t = [23 25 27; 23 25 27; 23 25 27]

demanda = demandaf/65
c = cf*65
h = hf*65

## Função objetivo 

producao = sum(c[i,j] * x[i,j] for i=1:P, j=1:T)
estoque = sum(h[i,j] * e[i,j] for i=1:P, j=1:T)


@objective(model, Min, producao + estoque)

## Restrição (1) - Conservação de estoque
for i=1:P, j=1:T
    if j==1
        @constraint(model, sum(x[i,j] - demanda[i,j]) == e[i,j])
    else   
        @constraint(model, sum(e[i, j-1] + x[i,j] - demanda[i,j] ) == e[i,j])
    end
end

## Restrição (2) - Capacidade de produção

for i=1:3
    @constraint(model, sum(x[i,j] * t[i,j] for j=1:3) <= disponibilidade[i])
end


##Restrição (3) - Garantia de atendimento à demanda

@constraint(model, e[1,3] == 0)
@constraint(model, e[2,3] == 0)
@constraint(model, e[3,3] == 0)
for i=1:P-1 , j=1:T-1
@constraint(model, e[i,j] >= 0)
end


print(model)
optimize!(model)


status = JuMP.optimize!(model)
status = termination_status(model)
println("STATUS: ", status, " ---------------------------")

pretty_table(value.(x))
pretty_table(value.(e))


println("-------------------------------------------")
println("plano em m2")
println("-------------------------------------------")
println("")
pretty_table(value.(x*65))
pretty_table(value.(e*65))
